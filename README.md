# OroApruveBundle

OroApruveBundle adds [integration](https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/IntegrationBundle) with [Apruve](https://apruve.com/) B2B Credit Management and Automation Platform in Oro applications.

The bundle allows admin users to enable and configure the Apruve [payment method](https://github.com/oroinc/orocommerce/tree/master/src/Oro/Bundle/PaymentBundle) available on the [checkout](https://github.com/oroinc/orocommerce/tree/master/src/Oro/Bundle/CheckoutBundle) process and therefore empower customers to pay for orders by credit obligations attested by Apruve.

## Table of Contents

 - [Technical Components](#technical-components)
 - [PaymentTransaction Lifecycle](#paymenttransaction-lifecycle)
 - [Things to Consider](#things-to-consider)
 - [FAQ](#faq)
 - [Links](#links)

## Technical Components:

1. Integration type:
 - `Oro\Bundle\ApruveBundle\Integration\ApruveChannelType`
 - `Oro\Bundle\ApruveBundle\Integration\ApruveTransport`
2. Payment method:
 - `Oro\Bundle\ApruveBundle\Method\ApruvePaymentMethod`
 - see namespace `Oro\Bundle\ApruveBundle\Method\PaymentAction` for concrete payment method actions implementations
3. Apruve-specific models and builders for them:
 - see namespaces `Oro\Bundle\ApruveBundle\Apruve\Model` and `Oro\Bundle\ApruveBundle\Apruve\Builder`
4. Apruve rest client:
 - `Oro\Bundle\ApruveBundle\Client\ApruveRestClient` - works with `RestClientFactoryInterface` under the hood
 - `Oro\Bundle\ApruveBundle\Client\Request\ApruveRequest` - request DTO
5. Apruve webhooks:
 - `Oro\Bundle\ApruveBundle\Controller\WebhookController` - entry point of webhooks processing.
 - `Oro\Bundle\ApruveBundle\EventListener\Callback\PaymentCallbackListener` - handles "return" payment event, which is triggered when user authorizes a payment. Delegates further processing to `Oro\Bundle\ApruveBundle\Method\PaymentAction\AuthorizePaymentAction`.


## PaymentTransactions Lifecycle:

1. PaymentTransaction 1 is created in `Oro\Bundle\ApruveBundle\Method\PaymentAction\PurchasePaymentAction` when a customer clicks Submit at the last step of checkout process.

2. If a customer authorizes payment in Apruve lightbox, PaymentTransaction 1 is updated with Apruve Order Id and is marked as successful in `Oro\Bundle\ApruveBundle\Method\PaymentAction\AuthorizePaymentAction`. Otherwise, nothing is changed.

3. Once a payment is authorized, it can be invoiced using "Send Invoice" button in admin area on Order view page in the "Payment History" section. When you click "Send Invoice", PaymentTransaction 2 is getting created along with Apruve Invoice in `Oro\Bundle\ApruveBundle\Method\PaymentAction\InvoicePaymentAction`. As soon as Apruve Invoice is created successfully, PaymentTransaction 3 is getting created along with Apruve Shipment entity in `Oro\Bundle\ApruveBundle\Method\PaymentAction\ShipmentPaymentAction`.

4. When a customer fulfils invoice, Apruve notifies about it via webhook `invoice.closed` which is getting processed starting from in `Oro\Bundle\ApruveBundle\Controller\WebhookController`. In case of success, PaymentTransaction 4 is created and marked as successful. If a customer does not pay for Apruve Invoice in time, it is marked as overdue and silently canceled on Apruve side.


## Things to Consider:

1. Apruve does not properly respect `price_total_cents` property of Apruve LineItem - it is not taken into account when secure hash is being generated from Apruve Order on Apruve side, though Apruve takes `amount_cents` (see [Merchant Integration Tutorial][1]). That's why it was decided (and approved by Apruve Support) to use `amount_cents` property in Apruve LineItem entity. See `Oro\Bundle\ApruveBundle\Apruve\Builder\LineItem\ApruveLineItemBuilder` and `Oro\Bundle\ApruveBundle\Apruve\Generator\OrderSecureHashGenerator` for details. On the other hand, `price_total_cents` is required for the line items which reside in Apruve Invoice, that is why both of these properties are present in Apruve LineItem entity.

2. Apruve wants to be notified about shipments from merchants who sell physical goods, but does not need this for other merchant types. Due to this fact it will not fulfil invoices in cases when it was not notified of shipment (when Shipment entity is not created via API) for merchants of physical goods. In order to unify the behavior for all types of merchants, it was decided to always notify Apruve about shipment, no matter goods of what type are sold. Shipment entity is created in Apruve along with Invoice entity when "Send Invoice" button is clicked on Order view page in the "Payment History" section.


## FAQ:

#### Is the customer forwarded to the Apruve-hosted payment page during the checkout process?

No, customers do not leave the commerce application during the checkout process. The user has to authorize payment in the Apruve popup (lightbox) at the last step of the checkout.

#### How can I login/register in Apruve sandbox?

You have to ask Apruve Support (support@apruve.com) for test merchant and buyer account.

#### I have a corporate Apruve account, but it is not accepted during checkout process

Corporate accounts differ. You should have a corporate account associated exactly with the merchant account you are trying to deal with.

Resources
---------

  * [OroCommerce Documentation](https://doc.oroinc.com)
  * [Contributing](https://doc.oroinc.com/community/contribute/)
  * [Apruve Sandbox][1]
  * [Apruve Docs][2]
  * [Apruve Guides][3]


[0]: https://www.apruve.com
[1]: https://test.apruve.com
[2]: https://docs.apruve.com/reference
[3]: https://docs.apruve.com/guides

Please refer first to [UPGRADE.md](UPGRADE.md) for the most important items that should be addressed before attempting to upgrade or during the upgrade of a vanilla Oro application.

The current file describes significant changes in the code that may affect the upgrade of your customizations.

## 3.1.4
### Changed
* In `Oro\Bundle\AuthorizeNetBundle\Controller\Frontend\PaymentProfileController::deleteAction` 
 (`oro_authorize_net_payment_profile_frontend_delete` route)
 action the request method was changed to DELETE. 
* In `Oro\Bundle\AuthorizeNetBundle\Controller\SettingsController::checkCredentialsAction` 
 (`oro_authorize_net_settings_check_credentials` route)
 action the request method was changed to POST. 

## 1.6.0 (2018-01-31)
[Show detailed list of changes](incompatibilities-1-6.md)

## 1.4.0 (2017-09-29)
[Show detailed list of changes](incompatibilities-1-4.md)

## 1.3.0 (2017-07-28)
[Show detailed list of changes](incompatibilities-1-3.md)
### Changed
- hide loading mask only when apruve popup is fully loaded
## 1.2.2 (2017-06-26)
### Fixed
- added ability to press on any element in iClickButtonInEmbed

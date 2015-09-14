### Requirements

Swift 1.2, iOS 8.0 and later

### How to use the library

__Cocoapods__

The framework is available on Cocoapods. The following needs to be added in the ```Podfile```:
```
use_frameworks!
pod 'CheckoutKit'
```

Then run ```pod install``` and the framework is ready to use.

__Carthage__

Add the following line to your ```cartfile```: ```github "CKOTech/checkoutkit-ios" >= 1.0.0```.
Then run ```carthage update```. It will download the latest version of the CheckoutKit framework. More information about Cartage is available [here](https://github.com/Carthage/Carthage).

__Manually__

Otherwise, the framework source code can be downloaded and manually added in your Xcode workspace.
Download or clone the [github repository](https://github.com/CKOTech/checkoutkit-ios). In your workspace, choose File > "Add files to ... " and select ```CheckoutKit/CheckoutKit.xcodeproj```.
Go to your project > General > Linked Frameworks and Libraries and add ```CheckoutKit.framework```.

### Example

A demo application is available in the folder *CheckoutKit/CheckoutKitExample*

Import the **CheckoutKit** framework in your code as below:
```
import CheckoutKit
```

You will need to specify at least the public key when creating an instance of ***CheckoutKit***. We offer a wide range of constructors, the ***CheckoutKit*** instance is entirely customizable:

```
CheckoutKit.getInstance(pk: String, env: Environment, debug: Bool, logger: Log, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(pk: String, debug: Bool, logger: Log, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(pk: String, env: Environment, logger: Log, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(pk: String, env: Environment, debug: Bool, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(pk: String, env: Environment, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(pk: String, logger: Log, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(pk: String, debug: Bool, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(pk: String, error: NSErrorPointer) -> CheckoutKit?
CheckoutKit.getInstance(error: NSErrorPointer) -> CheckoutKit? // to be called only once the CheckoutKit object has been initialized and a public key has been provided

```

These functions can return a nil object if an error occurred, in which case the NSErrorPointer contains useful details. If the public key is invalid, the error domain is set to CheckoutError.InvalidPK and if getInstance without the public key argument is called before the object is instantiated, the error domain is set to CheckoutError.NoPK (both contains custom error messages).
Here are more details about the parameters :
- pk : String containing the public key. It is tested against the following regular expression : ```^pk_(?:test_)?(?:\w{8})-(?:\w{4})-(?:\w{4})-(?:\w{4})-(?:\w{12})$```. Mandatory otherwise the ***CheckoutKit*** object cannot be instantiated.
- env : Environment object containing the information of the merchant's environment, default is SANDBOX. Optional.
- debug : Bool, if the debug mode is activated or not, default is true. Optional.
- logger : Log object printing information, warnings and errors to the console (for now). Optional.

If **debug** is set to true, many actions will be traced to the logging system.

The available functions of ***CheckoutKit*** can be found in the documentation : JAVADOC LINK.

Another class is available for utility functions: ***CardValidator***. It provides static functions for validating all the card related information before processing them. The list of the available functions is available in the documentation : JAVADOC LINK.


**Create card token**

```swift
import CheckoutKit

let publicKey: String = "your_public_key"

var error: NSError?
/* Instantiates the CheckoutKit object with the minimum parameters, default configuration for the other ones */
var ck = CheckoutKit.getInstance(publicKey, error: &error)

if ck == nil {
    /* Happens when the public key is not valid, raised during the instanciation of the CheckoutKit object */
    /* Type of the error in the enum CheckoutError. Different types are NoPK (if getInstance is called with no parameters and no public key has been provided before) and InvalidPK (if the public key is invalid) */
} else {

	/* Take the card information where ever you want (form, fields in the application page...) */
    let card = Card(name: nameField.text, number: numberField.text, expYear: yearField.text, expMonth: monthField.text, cvv: cvvField.text, billingDetails: nil, error: &error)

    if card == nil {
        /* Happens when the card informations entered by the user are not correct */
		/* Type of the error in the enum CardError. Different types are InvalidCVV (if the CVV does not have the correct format), InvalidExpiryDate (if the card is expired), InvalidNumber (if the card's number is wrong). */
    } else {
        ck!.createCardToken(card!, completion:{ (resp: Response<CardTokenResponse>) -> Void in
            if (resp.hasError) {
                /* Print error message or handle it */
            } else {
                /* The card object */
                let ct: CardToken = resp.model!.card
                /* The field containing the card token to make a charge */
                let cardToken: String = resp.model!.getCardToken()
            }
        })
    }
}
```

### Logging

Most of the activity of the **CheckoutKit** is logged either as information, warning or error. All the logs are made to the console for now. Logging occurs only if the debug mode is activated (true as default, but can be explicitely set to false). The printing format is ```date (yyyy/MM/dd HH:mm:ss)  **Subject/Class name**  logged message```. The logger can be modified via the functions setLogger or getInstance. The log entries are then added to the logger specified.

### Unit Tests

All the unit test written using XCTest and Nimble/Quick and resides in CheckoutKit/CheckoutKitTests.

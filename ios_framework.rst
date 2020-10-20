.. _ios_framework:

.. toctree::
   :maxdepth: 2
   :caption: Custom iOS App Guide


Custom iOS App Guide
====================
By default Activeconnect will send authentication requests to the reference Activeconnect mobile application.
Activeconnect Applications can be configured to send authentication requests to custom mobile applications.
In this case the custom application is responsible for:
* Registering the mobile device with Activeconnect
* Collecting the required authentication data in response to authentication requests.

The easiest way to provide this functionality in a custom iOS application is to integrate the iOS framework into the custom application.
The iOS framework can be downloaded from `GitHub. <https://github.com/Activeconnect/ActiveconnectiOS-Framework>`__


Installation
############
The easiest way to integrate Activeconnect into your application is to use `CocoaPods <https://cocoapods.org>`__.
To integrate ActiveconnectiOS into your xCode project using CocoaPods, specify it in your `podfile`.
```
pod 'ActiveconnectiOS', '~>0.1.0'
```

Reference
#########
The ActiveconnectiOS framework reference documentation can be found `here <https://activeconnect.github.io/iosdocs/>`__.

Getting Started
###############
Create an Activeconnect Account and Application
-----------------------------------------------
The first stage in integrating Activeconnect into your applicaton is to create an Activeconnect Application as described `here <https://activeconnect.readthedocs.io/en/latest/getting_started.html>`__.
**Save the application id and application secret for your Activeconnec Application you will need them to communicate with Activeconnect.**

Add the ActiveconnectiOS framework to your project
--------------------------------------------------
To integrate ActiveconnectiOS into your xCode project using CocoaPods, specify it in your `podfile`.
```
pod 'ActiveconnectiOS', '~>0.1.0'
```

Application Configuration
#########################
Enable Push Notifications
-------------------------
Activeconnect uses Push Notifications to notify users of Authentication Requests.
Add the `Push Notifications <https://developer.apple.com/documentation/usernotifications>`__ entitlement to your application.

Configure Activeconnect Notifications Credentials
-------------------------------------------------
In order to send Push Notifications to your application you need to configure your Activeconnect:
    * Download the APNS certificates for your application from the Apple Developer Portal.
    * Import the certificate and private key into KeyChain (both development and production keys).
    * Export the certificate and private key from KeyChain as .p12 files.
    * Open the `Activeconnect Console <https://activeconnect.activeapi.ninja>`__
    * Select your application and click the **Details** button.
    .. image:: ./images/applications.png
    * In the details page click the settings icon and select **Notification Settings**.
    .. image:: ./images/applicationdetails.png
    * On the Notifications Settings page click the **Edit** button for iOS.
    .. image:: ./images/notificationsettings.png
    * On the **Apple iOS Notification Settings** page select **Custom iOS** app from the dropdown.
    .. image:: ./images/customnotification.png
    * Update the credentials with your APNS certificate and Private Key.
    * Repeat the process for iOS Sandbox using your development credentials.

Activeconnect will now route your users authentication requests to your iOS application.

Applications Entitlements
-------------------------
Activeconnect use Camera and Bluetooth services on your user's mobile device.
You must add the following entries to your application's info.plist.
    .. image:: ./images/privacy.png

.. table:: info.plist

+-------------------------------------------+-----------------------------------------------------------------------+
| key                                       |     value                                                             |
+===========================================+=======================================================================+
|  NSBluetoothAlwaysUsageDescription        |   uses Bluetooth to verify proximity                                  |
+-------------------------------------------+-----------------------------------------------------------------------+
|  NSBluetoothPeripheralUsageDescription    |   uses Bluetooth to verify proximity                                  |
+-------------------------------------------+-----------------------------------------------------------------------+
|  NSCameraUsageDescription                 |   uses the camera to capture images required for facial recognition.  |
+-------------------------------------------+-----------------------------------------------------------------------+
|  NSFaceIDUsageDescription                 |   uses FaceID to verify that you can unlock your phone.               |
+-------------------------------------------+-----------------------------------------------------------------------+
|  NSPhotoLibraryUsageDescription           |   will not use any of your stored images.                             |
+-------------------------------------------+-----------------------------------------------------------------------+


Background Modes
----------------
Add the Background Modes entitlement to your application in xCode.
Enable the following modes:
    * Acts as a Bluetooth LE accessory
    * Remote Notifications
    .. image:: ./images/backgroundmodes.png


Key Concepts
############
When a system uses Activeconnect to authenticate its users:
    * A notification is sent to the users registered device.
    * The registered device collects the required information and send it to Activeconnect.
    * Activeconnect processes the collected data and makes an authentication decision.

A custom Activeconnect mobile device is responsible for:
    * Registering the user's device.
    * Processing notifications from Activeconnect.
    * Collecting the required data.

Registering Users
------------------
Activeconnect does not manage your users or replace your sign up flow.
You register your user with Activeconnect by providing a token that you can relate back to your user See :ref:`getting_started` for information about user management.
There are 2 common registration flows:
    * User registers using your mobile application.
    * User registers externally (in a web browser for example).

**You must wait until your application has completed the Push Notification registration process before registering devices**

Mobile Application Registration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The Activeconnect iOS framework provides an interface for registering users and their mobile device.
This example assumes that the mobile application has:
    * Registered for Push Notifications and stored the returned token in **NOTIFICATION_TOKEN**
    * Registered the user **user** with its own system.
    .. code-block:: swift

        import ActiveconnectiOS
        ...
        // user: client generated identifer for the client applications user.
        // NOTIFICATION_TOKEN: The token received by the application when it registers for Push Notifications
        // MY_APPLICATION_ID: The ID of the Activeconnect application (from Activeconnect Console)
        // MY_APPLICATION_SECRET: The Application Secret of the Activeconnect application (from Activeconnect Console)

        // Create an Activeconnect Management API instance using credentials for client application
        let management_api = ManagementAPI(application_id: MY_APPLICATION_ID,application_secret: MY_APPLICATION_SECRET)

        // Register the user with Activeconnect and register this device.
        management_api.create_user_and_register_device(user: user, notification_token: NOTIFICATION_TOKEN) {(user, device) in
            // user: The identifier of the user (same as passed to create_user_and_register_device)
            // device: ACMobileDevice that contains user and device information required for subequent Activeconect calls.

            // This example encodes the ACDeviceData as JSON and stores it in user defaults.
            let encoder = JSONEncoder()

            if let encoded = try? encoder.encode(device){
                let defaults = UserDefaults.standard
                defaults.set(encoded, forKey: "user_data")
                defaults.set(user, forKey: "user_name")
            }
        } failure: { (user, error) in
            // user: The identifier of the user (same as passed to create_user_and_register_device)
            // error: Error that describes failure reason.
        }

External Registration
^^^^^^^^^^^^^^^^^^^^^
Activeconnect generates custom registration links for associating a device with a user.
The external client application can get the registration link and share it with the mobile application.
It is the client's responsibility to share these links with the mobile application.
Alternatively, the mobile application can provide an interface for the user to enter their username and obtain a registration link.
The Activeconnect iOS framework provides a method to get a registration link for the user
    .. code-block:: swift

        import ActiveconnectiOS
        ...
        // user_id: client generated identifer for the client applications user.
        // display_name: A readable name for the user (user_id may be a random token)
        // MY_APPLICATION_ID: The ID of the Activeconnect application (from Activeconnect Console)
        // MY_APPLICATION_SECRET: The Application Secret of the Activeconnect application (from Activeconnect Console)


        // Create an Activeconnect Management API instance using credentials for client application
        let management_api = ManagementAPI(application_id: MY_APPLICATION_ID,application_secret: MY_APPLICATION_SECRET)

        management_api.get_registration_link(user_id: user, display_name: user) { (reg_link) in
                    // reg_link: URL that can be used to register the device.
                } failure: { (error) in
                    // Failed to get registration link.
                }
            }

Whichever method is used to obtain the registration link, the link can now be used to register the device
    .. code-block:: swift

        import ActiveconnectiOS
        ...
        ACMobileDevice.registerDevice(identifier: user_id, registration_link: reg_link, notification_token: NOTIFICATION_TOKEN) { (identifier, device) in
                    // Device registered
                    // identifier: identifier for user (same as value passed to registerDevice)
                    // device: ACMobileDevice that contains user and device information required for subequent Activeconect calls.
                    print("registered")

                } failure: { (identifier, error) in
                    // identifier: identifier for user (same as value passed to registerDevice)
                    // error: Error indicating failure reason
                    print("failed")
                }

Storing Device Information
^^^^^^^^^^^^^^^^^^^^^^^^^^
The client application should store the ACMobileDevice instance returned by device registration.
ACMobileDevice implements the Codeable interface and can be persisted using Swift encoding (JSONEncoder for example).
The iOS UserDefaults can be used to store the data, however the iOS KeyChain main be a more secure option.

.. _ios-training-label:
Training
--------
After a device has been registered Activeconnect may have to collect some training data for the device.
After registration check the `training_required` property of `ACMobileDevice` to determine if Activeconnect needs to perform training.
    .. code-block:: swift

        if registeredDevice.training_required{
            // Perform training...
        }

The Activeconnect iOS framework provides a default UI for performing training.
The example code below shows a view controller that presents the training UI.

    .. code-block:: swift

        // Class implements ACTrainingDelegate, which processes training results.
        class MainViewController : UIViewController, ACTrainingDelegate{

            func doTraining()->Void{
                let device = self.get_device()
                let user_id = self.get_user_id()

                var training_vc = ACTrainingViewController()
                training_vc.device = device
                training_vc.identifier = user_id
                training_vc.delegate = self

                // Present the view controller
                self.present(training_vc, animated: true, completion: nil)
            }

            func get_device()->ACMobileDevice{
                // Return registered device information
            }

            func get_user_id()->String{
                // Return the user id
            }

            func save_device(user: Any?, device: ACDeviceData)->Void{
                // Store the update device data.
            }

        }
        // Implementation of ACTrainingDelegate
        extension MainViewController: ACTrainingDelegate{
            // Training is complete, update the stored device information
            func didCompleteTraining(identifier: Any?, device: ACMobileDevice) {
                print("completed training")
                self.save_device(...)
            }

            func didFailToCompleteTraining(identifier: Any?, device: ACMobileDevice, error: Error?) {
                print("training failed")
            }
        }

If you prefer to use Storyboards and Segues you can create a new ACTrainingViewController derived class and instantiate an instance in Interface Builder.


Updating Device Information
---------------------------
Every time your application starts up or the user changes Notification Settings, you should update the device stored ACMobileDevice.
Use the `update` method of `ACMobileDevice` to update the device information.
When the device is updated Activeconnect may need to perform training. See :ref:`training <ios-training-label>` for information about training.
    .. code-block:: swift

        // Class implements ACTrainingDelegate, which processes training results.
        class MainViewController : UIViewController, ACTrainingDelegate{

            // Update the stored mobile device.
            func updateDevice()->Void{
                let device = self.get_device()
                let user_id = self.get_user_id()

                device.update(identifier: user_id, notification_token: NOTIFICATION_TOKEN) { (identifier, updated_device) in
                    print("Updated Device")

                    // Save the device
                    self.save_device(identifier: user_id, device: device)
                    if mobile_device.training_required{
                        DispatchQueue.main.async {
                            self.doTraining()
                        }
                    }
                } failure: { (identifier, error) in
                    print("Update error")
                }

            }

        }

Handling Authentication Requests
--------------------------------
Activeconnect delivers authentication requests to mobile applications using Push Notifications.
The notification title and description use string IDs so the client application must have the following strings in its string resource.

    .. code-block:: swift

        /*
          Localizable.strings
            ...
        */
        ...
        /*Content of the login notification*/
        "NOTIFICATION_TITLE" = "Activeconnect® AuthenticationRequest";
        "NOTIFICATION_BODY" = "TAP TO ACCEPT or clear to cancel.";
        "ACTION_LOGIN_ALERT_NOTIFICATION" = "VERIFY";

You can change the values of the strings but the keys must exist.

When a mobile application receives a Push Notification it should pass it to the Activeconnect iOS framework,
which will decode the payload and carry out the required authentication steps.

    .. code-block:: swift

        var notificationHandler: ACNotificationHandler?


        class AppDelegate: UIResponder, UIApplicationDelegate {

            func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

                self.notificationHandler = ACNotificationHandler(delegate: self)
                return true
            }

            // Client implemented function that gets any device information stored on this device.
            func get_registered_devices()->[ACMobileDevice]{

            }

            // Application has received a Push Notification.
            func application(   _ application: UIApplication,
                                didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                                fetchCompletionHandler completionHandler:
                                @escaping (UIBackgroundFetchResult) -> Void) {

                // Get the device information stored on this device
                let registeredDevices: self.get_registered_devices()

                // Has the notification handler been initialized?
                if let notificationHandler = self.notificationHandler{

                    // Pass the notification to the notification handler along with the list of registered devices.
                    // The notification handler will check that the notification is for the device.
                    // The notification handler will call either presentAuthenticationUI or sessionStatusChanged
                    // methods of its delegate if it handles the notification.
                    let activeconnectResponse = notificationHandler.processNotification(userInfo: userInfo, registeredDevices: registeredDevices)

                    if activeconnectResponse.activeconnectNotification{
                        completionHandler(activeconnectResponse.suggestedCompletionResult)
                        return
                    }else{
                        // This is not an Activeconnect notification so continue regular notification handling.
                    }
                }
                // Perform regular notification handling.
                completionHandler(.noData)
            }
        }

        extension AppDelegate: ACNotificationHandlerDelegate{
            func sessionStatusChanged(status: ACSessionStatus) {
                print("session status has changed")
            }

            func presentAuthenticationUI(request: ACAuthenticationRequest, device: ACMobileDevice) {
                // Present the authentication flow.
            }
        }

Presenting the Authentication UI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The Activeconnect iOS framework provides the class `ACAuthenticationViewController` to display the authentication UI.
The `ACNotificationHandler.presentAuthenticationUI` should create an instance of `ACAuthenticationViewController`,
set the `authenticationRequest` and `mobileDevice` properties and display the view controller.

In iOS applications, notifications are received by the AppDelegate but there is no 'recommended' way of presenting a
ViewController from the AppDelegate.

When the `ACAuthenticationViewController` completes it will call either the `didAuthenticate` or `didFailToAuthenticate` member
of its `authenticationDelegate`. The client application is responsible for dismissing the `ACAuthenticationViewController`.

    .. code-block:: swift

        class MainViewController: UIViewController{
            ...
            func presentAuthenticationUI( request: ACAuthenticationRequest, device: ACMobileDevice ) -> Void{
                let authenticationController = ACAuthenticationViewController()
                authenticationController.authenticationRequest = request
                authenticationController.mobileDevice = device
                authenticationController.authenticationDelegate = self
                authenticationController.modalPresentationStyle = .fullScreen
                self.present(authenticationController, animated: true, completion: nil
            }
        }

        extension MainViewController: ACAuthenticationDelegate{
            func didAuthenticate() {
                print("authenticated")
                // return UI to previous state.
            }

            func didFailToAuthenticate(error: Error?) {
                print("failed to authenticate")
                // return UI to previous state.
            }
        }

You can create a ACAuthenticationViewController in a StoryBoard or Nib file rather than creating it programatically.

Logging
-------
The Activeconnect iOS framework uses system logging to display diagnostic information.
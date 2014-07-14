# Apple Push Notification (APNS) Setup Guide #

Guide to setup APNS.

## Create Certificates, Identifiers and Profiles ##

Log into the [Apple Developer Member Center](https://developer.apple.com/account/overview.action).

### Create the App Identifier ###

  * Go to "Identifiers" -> "App IDs" and create a new one
  * Enter the name of your app and an explicit bundle ID
  * Ensure "Push Notifications" is checked under "App Services"
  * Click continue and submit to save the app identifier

Note: It may make sense to use separate bundle identifiers for development and production to allow devices to run both  versions simultaneously.

### Certificates ###

This guide uses the openssl command rather than Keychain Access because it's easier to script and works cross-platform.

  * Open up a terminal and enter the following line to setup your app name. This is a variable that will be used by the other commands in this section to help generate consistent filenames so please adapt it to fit your needs:

    ```
    APP="YourApp-Development-Push-Sandbox"
    ```

  * Create the private key and associated certificate signing request (CSR):

    ```
    openssl req -new -newkey rsa:2048 -nodes -out "$APP.certSigningRequest" -keyout "$APP.key.pem"
    ```

    Fill out your location information, the OU, Email Address, Challenge Password, and CN may all be left blank. Skip entering the export password.

  * Create the APNS sandbox certificate for development use:

    * After you've created the identifier, go to "Certificates" and add a new one
    * Select "Apple Push Notification service SSL (Sandbox)" and click continue
      * Note: In addition to the push certificate, you'll also need to generate a separate key, csr, and cert for signing the app for development and distribution with if you haven't already done so
    * Now select the App ID that corresponds to the certificate you're creating and click continue
    * In the next step, choose the CSR file you created
    * The last step will allow you to download the generated certificate, named "aps_development.cer" for the sandbox and "aps_production.cer" for production
    * Rename the certificate to match your key and CSR (optional but recommended):
    
    ```
    mv ~/Downloads/aps_development.cer "$APP.cer"
    ```
    
    * Create the P12 and PEM files with the private key and signed certificate:

    ```
    openssl x509 -in "$APP.cer" -inform der -outform pem -out "$APP.cert.pem"
    cat "$APP.cert.pem" "$APP.key.pem" > "$APP.pem"
    openssl pkcs12 -export -in "$APP.pem" -out $APP.p12 -name "$APP"    
    ```
    
    You'll must specify an export password when generating the pkcs12 file.

#### Create the production APNS certificate ####

In the above steps you created a key and CSR for the sandbox APNS. To create an adhoc/production version:

  * Follow the above steps
  * When generating the certificate, select "Apple Push Notification service SSL (Production)" instead of the Sandbox option
  * You should replace "Sandbox" with "Production" to know what type of push service each certificate is for

### Setup production push service for the App Identifier ###

  * After creating the identifier, you'll need to setup the APNS certificate for production push notifications, to do this click on the production/ad-hoc App ID you created and click "Edit"
  * Click "Create Certificate" under "Production SSL Certificate"
  * Use the certificate signing request that you generated in the previous section

### Devices ###

  Go to "Devices" and add the device ID (UDID) for any devices you'd like to test with

### Provisioning Profiles ###

* Go to "Provisioning Profiles" and add a new one
* Select "Ad Hoc" for the profile type (you will also want to create a provisioning profile for development purposes)
* Select the Ad Hoc App ID you created
* Select your signing certificate for distribution
  * Note: This is a different type of certificate than the APS certificates generated in this guide, but the OpenSSL steps to generate it (above) are the same
* Select the devies you want to allow
* Enter the profile name: YourApp Ad Hoc
* Download the provisioning profile file and open it, which will import the file into "~/Library/MobileDevice/Provisioning Profiles/UUID.mobileprovision"

## Final Steps ##

After following the above steps, you should files that are something similar to the following:

* YourApp-Development-Push-Production.cer
* YourApp-Development-Push-Production.cert.pem
* YourApp-Development-Push-Production.certSigningRequest
* YourApp-Development-Push-Production.key.pem
* YourApp-Development-Push-Production.p12
* YourApp-Development-Push-Production.pem

* YourApp-Development-Push-Sandbox.cer
* YourApp-Development-Push-Sandbox.cert.pem
* YourApp-Development-Push-Sandbox.certSigningRequest
* YourApp-Development-Push-Sandbox.key.pem
* YourApp-Development-Push-Sandbox.p12
* YourApp-Development-Push-Sandbox.pem

* YourApp-Development.mobileprovision
* YourApp-AdHoc.mobileprovision

### Signing ###

In addition to the above files, you'll want to keep track of your own iOS signing certificates which are split out into development and production versions. For development, you can generally re-use these certifiates across all of the apps in your account but it may be advisable to create separate production signing certificates for App Store releases.

You may generate your signing certificates using the same openssl-based steps above for creating the push certificates. Note: If you used the above commands, you'll need to import the .cer and .p12 files into your keychain in order for Xcode to be able to sign your apps.

### Testing ###

To test connecting to the APNS gateway (the development sandbox in this case):

```bash
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert YourApp-Development-Push.cert.pem -key YourApp-Development-Push.key.pem
```

## APNS with over-the-air (OTA) Deployments ##

To get push notifications working with over-the-air builds, you need to create and use a production certificate. (The devleopment certificates are intended to be used when installing your app directly from Xcode/organizer and will provide extra debug information.)

## Contact ##

- Brian Stanback https://github.com/stanback
- Daniel Khamsing https://github.com/dkhamsing

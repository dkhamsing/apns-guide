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

  * Create the private key and associated certificate signing request (CSR):

    ```
    APP=YourApp-Development-Push
    openssl req -new -newkey rsa:2048 -nodes -out $APP.certSigningRequest -keyout $APP.key.pem
    ```

    Fill out your location information, the OU, Email Address, Challenge Passowrd, and CN may all be left blank. Skip entering the export password.

  * Create the development certificate:

    * After you've created the identifier, go to "Certificates" and add a new one
    * Select "Apple Push Notification service SSL (Sandbox)" and click continue
      * Note: In addition to the push certificate, you'll also need to generate a separate key, csr, and cert for signing the app for development and distribution with if you haven't already done so
    * Now select the App ID that corresponds to the certificate you're creating and click continue
    * In the next step, choose the CSR file you created
    * The last step will allow you to download the generated certificate, named "aps_development.cer" by default
    
    * Rename the certificate to match your key and CSR (the filename should either start with ios or aps and end with development, production, or distribution):
    
    ```
    APP=YourApp-Development-Push
    mv aps_development.cer $APP.cer
    ```
    
    * Create the P12 and PEM files with the private key and signed certificate:

    ```
    APP=YourApp-Development-Push
    openssl x509 -in $APP.cer -inform der -outform pem -out $APP.cert.pem
    cat $APP.cert.pem $APP.key.pem > $APP.pem
    openssl pkcs12 -export -in $APP.pem -out $APP.p12     
    ```

You should replace "Development" with "AdHoc", "Production" or something meaingful depending on what the certificate is going to be used for.

#### Create the APN certificate ####

Todo

#### Create the production/ad-hoc certificate ####

  * Create an additional key and CSR
  * Go to "Certificates" and add another one; repeat the steps in this section and select "Apple Push Notification service SSL (Production)" instead of the Sandbox option

By the end of the certificate generation process, you should have two certifiates for development: "iOS App Development" for signing and "Apple Push Notification service SSL (Sandbox)" for APN connections. For production, you should have: "App Store and Ad Hoc" for signing, and two other certificates for APN connections: "Apple Push Notification service SSL (Development)" and "Apple Push Notification service SSL (Production)".

### Setup production push service for the App Identifier ###

  * After creating the identifier, you'll need to setup the APNS certificate for push notifications, to do this click on the production/ad-hoc App ID you created and click "Edit"
  * Click "Create Certificate" under "Development SSL Certificate"
  * Repeat the steps in the section above to generate a new private key and CSR

### Devices ###

  Go to "Devices" and add the device ID (UDID) for any devices you'd like to test with

### Provisioning Profiles ###

* Go to "Provisioning Profiles" and add a new one
* Select "Ad Hoc" for the profile type
* Select the Ad Hoc App ID you created
* Select your distribution certificate
  * This is a different type of certificate than the APS certificates generated in this guide, but the openssl steps to generate it are the same
* Select the devies you want to allow
* Enter the profile name: YourApp Ad Hoc
* Download the provisioning profile file

## Finishing up and Testing ##

After following the above steps, you should files that are something similar to the following:

* YourApp-AdHoc-Push-Production.cer
* YourApp-AdHoc-Push-Production.cert.pem
* YourApp-AdHoc-Push-Production.certSigningRequest
* YourApp-AdHoc-Push-Production.key.pem
* YourApp-AdHoc-Push-Production.p12
* YourApp-AdHoc-Push-Production.pem
* YourApp-AdHoc-Push.cer
* YourApp-AdHoc-Push.cert.pem
* YourApp-AdHoc-Push.certSigningRequest
* YourApp-AdHoc-Push.key.pem
* YourApp-AdHoc-Push.p12
* YourApp-AdHoc-Push.pem
* YourApp-AdHoc.mobileprovision

To test connecting to the APNS gateway (the development sandbox in this case):

```bash
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert YourApp-Development-Push.cert.pem -key YourApp-Development-Push.key.pem
```

## APNS with over-the-air (OTA) Deployments ##

To get push notifications working with over-the-air builds, you need to create and use a production certificate. (The devleopment certificates are intended to be used when installing your app directly from Xcode/organizer and will provide extra debug information.)

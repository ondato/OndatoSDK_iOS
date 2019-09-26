# Ondato iOS SDK

## Table of contents

* [Overview](#overview)
* [Getting started](#getting-started)
* [Handling callbacks](#handling-callbacks)
* [Customising SDK](#customising-sdk)


## Overview

This SDK provides a drop-in set of screens and tools for iOS applications to allow capturing of identity documents and face photos/live videos for the purpose of identity verification. The SDK offers a number of benefits to help you create the best onboarding/identity verification experience for your customers:

- Carefully designed UI to guide your customers through the entire photo/video-capturing process
- Modular design to help you seamlessly integrate the photo/video-capturing process into your application flow
- Advanced image quality detection technology to ensure the quality of the captured images meets the requirement of the Ondato identity verification process, guaranteeing the best success rate
- Direct image upload to the Ondato service, to simplify integration\*

\* Note: the SDK is only responsible for capturing and uploading photos/videos. You still need to access the [Ondato API](https://api.ondato.com/verifid/index.html) to create and manage checks.

## Important note

We recommend you to lock your app to a portrait orientation.

## Getting started

- SDK supports iOS 10.0
- SDK supports Swift 5

### 1. App permissions

The Ondato SDK makes use of the device Camera. You will be required to have the `NSCameraUsageDescription` key in your application's `Info.plist` file, also need to provide value for `NSPhotoLibraryUsageDescription` key:
```
<key>NSCameraUsageDescription</key>
<string>Required for document and facial capture</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Framework uses</string>
```
### 2. Installation 

### CocoaPods

[CocoaPods](https://cocoapods.org) is a dependency manager for Cocoa projects. For usage and installation instructions, visit their website. To integrate OndatoSDK into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
pod 'OndatoSDK'
```

Install pod and build first time.

Sign subframeworks. Open `Build Phases` tab of your target and select add `New Run Script Phase`. Past code below to scipt.
```
pushd "${TARGET_BUILD_DIR}"/"${PRODUCT_NAME}".app/Frameworks/OndatoSDK.framework/Frameworks

for EACH in *.framework; do
/usr/bin/codesign --force --deep --sign "${EXPANDED_CODE_SIGN_IDENTITY}" --entitlements "${TARGET_TEMP_DIR}/${PRODUCT_NAME}.app.xcent" --timestamp=none $EACH
done
popd
echo "BUILD DIR ${TARGET_BUILD_DIR}"

```

### 3. Creating the SDK configuration

(Create an OndatoService with initialize parameters using your username, along with the password, choose mode of :"test" and "live" environment, default mode is test. Also you can initialize SDK with token, token parameter is availble only for LIVE mode.) 

#### Swift

```swift
OndatoService.shared.initialize(username: "username", password: "password")
OndatoService.shared.initialize(username: "username", password: "password", token: "token") //'token' is optional
OndatoService.shared.serverMode = OndatoServerMode.test
```

### 4. Starting the flow

#### Swift


```swift
let sdk = OndatoService.shared.instantiateNavigationController()
sdk.modalPresentationStyle = .fullScreen
self.present(sdk, animated: true, completion: nil) //`self` should be your view controller
```

Congratulations! You have successfully started the flow. Carry on reading the next sections to learn how to:

- Handle callbacks
- Customise the SDK

## Handling callbacks

To handle result your view controller should implement `OndatoFlowDelegate` method `ondatoSDKDidFinished`:

```swift
OndatoService.shared.flowDelegate = {delegate : OndatoFlowDelegate}

func ondatoSDKDidFinished() {
        print("SDK closed.")
    }
```

## Customising SDK

### 1. Splash Screen Step
This screen is used to load data from server. Logo can be changed with the following parameter:`OndatoService.shared.logoImage = UIImage() // Your logo image`

### 2. Theme customisation

```swift
OndatoService.shared.appearance.progressColor = UIColor.orange //Defines the background color of the `ProgressBarView` which guides the user through the flow
OndatoService.shared.appearance.buttonColor = UIColor.orange //Defines the background color of the primary action buttons
OndatoService.shared.appearance.buttonTextColor = UIColor.white //Defines the background color of the primary action buttons text
OndatoService.shared.appearance.errorColor = UIColor.orange //Defines the background color of the error message background
OndatoService.shared.appearance.errorTextColor = UIColor.white //Defines the background color of the error message text color
OndatoService.shared.appearance.regularFontName = "Rambla-Regular" //Defines the regular text font 
OndatoService.shared.appearance.mediumFontName = "Rambla-Bold" //Defines the medium text font
OndatoService.shared.appearance.shouldShowSuccessWindow = false //Enables/disables success window
OndatoService.shared.appearance.shouldShowStartWindow = false //Enables/disables start window
```

### 3. Flow

`OndatoFlowDelegate` provide some optional methods to replace Ondato windows with custom ones:

```swift

func viewControllerForSplashScreen() -> UIViewController? {
        let vc = YourSplashScreenViewController()
        return vc
    }

func viewControllerForStart(_ startPressed: @escaping (() -> ())) -> UIViewController? {
    let vc = YourStartViewController()
    vc.startPressed = startPressed
    return vc
}

func viewControllerForDocumentType(availableTypes: NSArray, documentTypeSelected: @escaping ((OndatoDocumentType) -> ())) -> UIViewController? {
    let vc = YourChooseViewController
    vc.selected = documentTypeSelected
    vc.types = availableTypes as? [OndatoDocumentType]
    return vc
}

func viewForLoading(progress: Float) -> UIView? {
    let vc = YourLoadingViewController
    return vc.view // or any custom view
}
```

### 4. Localisation

Ondato iOS SDK already comes with out-of-the-box translations for the following locales:
- English (en) :uk:
- Lithuanian (lt) :lt:

```swift
OndatoLocalizeHelper.language = OndatoLanguage.EN // OndatoLanguage.LT
```

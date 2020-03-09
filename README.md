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

### Manually
Download archive `OndatoSDK.zip` from latest sdk releases. Add frameworks to your project:
1) ZoomAuthentication.framework (select `Ember & Sign`)
2) OndatoSDK.framework (select `Ember & Sign`)

Remove unnecessary architectures, needed for release. Open `Build Phases` tab of your target and select add `New Run Script Phase`. Paste code below to scipt.
```
# skip if we run in debug
if [ "$CONFIGURATION" == "Debug" ]; then
echo "Skip frameworks cleaning in debug version"
exit 0
fi

APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"

# This script loops through the frameworks embedded in the application and
# removes unused architectures.
find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
do
FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"

EXTRACTED_ARCHS=()

for ARCH in $ARCHS
do
echo "Extracting $ARCH from $FRAMEWORK_EXECUTABLE_NAME"
lipo -extract "$ARCH" "$FRAMEWORK_EXECUTABLE_PATH" -o "$FRAMEWORK_EXECUTABLE_PATH-$ARCH"
EXTRACTED_ARCHS+=("$FRAMEWORK_EXECUTABLE_PATH-$ARCH")
done

echo "Merging extracted architectures: ${ARCHS}"
lipo -o "$FRAMEWORK_EXECUTABLE_PATH-merged" -create "${EXTRACTED_ARCHS[@]}"
rm "${EXTRACTED_ARCHS[@]}"

echo "Replacing original executable with thinned version"
rm "$FRAMEWORK_EXECUTABLE_PATH"
mv "$FRAMEWORK_EXECUTABLE_PATH-merged" "$FRAMEWORK_EXECUTABLE_PATH"

done
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

To handle result your view controller should implement `OndatoFlowDelegate` methods `onSuccess` and `onFailure` which contains `OndatoError` {`CANCELED`, `BAD_SERVER_RESPONSE`}:

```swift
OndatoService.shared.flowDelegate = {delegate : OndatoFlowDelegate}

func onSuccess() {
    print("SDK is finished successfully.")
}
func onFailure(error: OndatoError) {
    print("Ondato error.")
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
OndatoService.shared.appearance.shouldShowSplashWindow = false //Enables/disables splash window
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

To override Zoom strings, add `ZoomLT.bundle` and `ZoomEN.bundle` to the project directory. Make sure to check "Copy items if needed" and select project target. Example bundles can be found in Releases `Localization.zip`.
Implementation:

```swift
if let bundleURL = Bundle.main.url(forResource: "ZoomLT", withExtension: "bundle"), let bundle = Bundle(url: bundleURL) {
    OndatoService.shared.zoomLocalizationBundleLT = bundle
}
        
if let bundleURL = Bundle.main.url(forResource: "ZoomEN", withExtension: "bundle"), let bundle = Bundle(url: bundleURL) {
    OndatoService.shared.zoomLocalizationBundleEN = bundle
}
```

To override Ondato strings, add `OndatoLT.strings` and `OndatoEN.strings` files to the project directory. Make sure to check "Copy items if needed" and select project target. Example files can be found in Releases `Localization.zip`.
Implementation:

```swift
if let fileURL = Bundle.main.url(forResource: "OndatoEN", withExtension: "strings") {
    OndatoService.shared.ondatoLocalizationFileEN = fileURL
}
        
if let fileURL = Bundle.main.url(forResource: "OndatoLT", withExtension: "strings") {
    OndatoService.shared.ondatoLocalizationFileLT = fileURL
}
```

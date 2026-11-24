# README #

## Requirements

* Android 5.0 (API 21) or newer (`minSdk 21`).
* The SDK adds these permissions to your app's manifest: `INTERNET`, `CAMERA` and `RECORD_AUDIO` (for the KYC steps). It asks the user for the camera permission itself when a KYC step needs it.
* The SDK uses Koin. If your app uses Koin too, nothing is needed: the SDK adds its modules to your Koin and doesn't change your own definitions.

## Installation

1. Download FormBuilderSDK.aar and its build.gradle file.
2. Change the project structure of your app from Android to Project.
3. Create a new folder in the same directory where your 'app' module is located and name it FormBuilderSDK (same as the .aar).
4. Paste the downloaded files there.
5. In the project level settings.gradle file, add include(":FormBuilderSDK") under include(":app") or include(":app", ":FormBuilderSDK")
6. In the app module build.gradle, add the following line in the dependencies section implementation(project(":FormBuilderSDK"))
7. Check the app module build.gradle (from step 6) if you have the following dependencies in your project:\
   implementation("androidx.core:core-ktx:1.13.1")\
   implementation("androidx.appcompat:appcompat:1.7.0")\
   implementation("com.google.android.material:material:1.12.0")\
   implementation("androidx.constraintlayout:constraintlayout:2.1.4")\
   implementation("androidx.recyclerview:recyclerview:1.1.0")\
   implementation("io.insert-koin:koin-android:3.5.6")\
   implementation("com.squareup.retrofit2:retrofit:2.9.0")\
   implementation("com.squareup.retrofit2:converter-gson:2.9.0")\
   implementation("com.google.code.gson:gson:2.8.6")\
   implementation("com.squareup.okhttp3:logging-interceptor:4.9.2")\
   implementation("com.vanniktech:android-image-cropper:4.7.0")\
   implementation("androidx.browser:browser:1.8.0")\
   implementation("com.github.bumptech.glide:glide:4.12.0")\
   If not please add them.
8. Sync the project and do a clean rebuild
9. Now you can proceed to the "How to use ?" section


## How to use ?

```kotlin
val intent = Intent(this, StartFormBuilderActivity::class.java)
intent.putExtra("processID", "PUT_PROCESS_ID_HERE")
intent.putExtra("callingServiceToken", "PUT_SERVICE_TOKEN_HERE")
startActivity(intent)
```
This creates an instance of FormBuilder and then opens a new interface based on the type of the process corresponding to the provided PROCESS_ID.
The callingServiceToken argument is a unique client identifier that will be stored in the submission. Calling service token can be null.

```kotlin
intent.putExtra("stepID", "PUT_STEP_ID_HERE")
```
This creates an instance of FormBuilder and then opens a new interface of a previously created process at the step which the user has reached before (e.g. if they have interrupted it for some reason).
It can also just be the first step for a process that the backend has preconfigured in some way. Either put processID or stepID when creating the intent.

```kotlin
intent.putExtra("baseURL", "PUT_BASE_URL_HERE")
```
You can put your own base url, to which you can send your requests, or you can simply not add it and use the default url (`https://kyc.rbbt.dev/ss/`). The URL must end with `/`.

To show a cancel button on the KYC (camera) screen, set `showCloseButton = true` in the `FBAppearance` you pass (see [Appearance](#appearance)). Default is `false`.

All intent extras:

| Extra | Type | Description |
|---|---|---|
| `processID` | String | Starts a new submission of this process. Use either `processID` or `stepID`. |
| `stepID` | String | Opens an existing submission at its current step. |
| `callingServiceToken` | String? | Your client identifier, stored in the submission. Optional. |
| `baseURL` | String | Backend URL, ending with `/`. Optional. |
| `appearance` | FBAppearance | Appearance of the SDK (see below). Optional. |
| `showCancelButton` | Boolean | **Deprecated**, use `FBAppearance.showCloseButton`. Still read when the appearance doesn't set `showCloseButton`. |

### Results

Register the callbacks before you start the SDK. Each callback can be registered once: registering again replaces the previous one.

```kotlin
FBDocDataFragment.ProcessResult.getProcessResultCallback { success ->
   if (success)
   //Do something on success
   else
   // Do something on failed process
}
```
Tells you whether the KYC (camera) step finished successfully or failed. It is called only for processes with a KYC step.

```kotlin
FormFragment.ProcessResult.postToHostUI { message ->
   // Use the value sent by the form
}
```
Sends you a value from the process: when a form step contains a field whose variable label (or special type) is `__postToHostUI`, you receive that field's default value.

### Navigation

* When a form step contains a `BACK` field, the SDK shows a back arrow at the top of the screen. It asks the user to confirm ("Are you sure you want to go back? You will lose the step information.") and then opens the BACK field's step, with the answers given there before already filled in.
* The system back button on a form step closes the SDK. If the user has entered something, it asks first ("Are you sure you want to close the form?").

### Language

The SDK's own texts (buttons, error messages, confirmations) are in English and Bulgarian and follow your app's language; the default is English. The backend receives the device language in the `Accept-Language` header, so texts that come from the process follow it too.

## Appearance

Optionally, you can pass an FBAppearance object in the intent.putExtra when starting the FormBuilder SDK, with which you can configure the appearance of the labels, fields, buttons:

```kotlin
val appearance = FBAppearance(
   backgroundColor = ContextCompat.getColor(this, R.color.teal_700),            // Background of all screens in the SDK (and the status bar)
   statusBarTextDark = false,                                                   // Color for the status bar text (false - white, true - black)
   textColor = ContextCompat.getColor(this, R.color.purple_500),                // Color of all text in the SDK

   toastFont = R.font.momoregular,                                              // Font of the directions text during KYC
   toastTextSize = 16,                                                          // Text size of the directions text during KYC
   toastTextColor = ContextCompat.getColor(this, R.color.white),                // Text color of the directions text during KYC

   consentTextSize = 16,                                                        // Text size of the consent view checkbox text
   textFont = R.font.momoregular,                                               // Font of the text that is displayed only (not editable text)
   viewTextSize = 16,                                                           // Text size of the text that is displayed only (not editable text)

   inputFont = R.font.momoregular,                                              // Font of the input text
   inputTextSize = 16,                                                          // Size of the input text
   inputTextColor = ContextCompat.getColor(this, R.color.white),                // Color of input text field
   inputBackgroundColor = ContextCompat.getColor(this, R.color.purple_500),     // Background color of editable text field

   buttonFont = R.font.momoregular,                                             // Font of the button text
   buttonTextSize = 16,                                                         // Text size of the button text
   buttonCornerRadius = 30f,                                                    // Corner radius of buttons (0f for square button, anything above 0f makes corners rounded)
   buttonTextColor = ContextCompat.getColor(this, R.color.color_blue),          // Color of the button text
   buttonBackgroundColor = ContextCompat.getColor(this, R.color.myColor),       // Background color of buttons

   progressBarColor = ContextCompat.getColor(this, R.color.teal_700),           // Color of the circular progress bar that appears when changing between screens or loading requests

   linearProgressIndicatorColor = ContextCompat.getColor(this, R.color.white),  // Color of the indicator that is loading while scanning your ID in KYC
   linearProgressTrackColor = ContextCompat.getColor(this, R.color.myColor)     // Color of the track that is shown while scanning your ID in KYC
)

val intent = Intent(this, StartFormBuilderActivity::class.java)
intent.putExtra("processID", "PUT_PROCESS_ID_HERE")
intent.putExtra("appearance", appearance)
startActivity(intent)
```
If you don’t pass an appearance object or a certain property, the SDK will use default values. Default text and background colors follow the device's light/dark mode. Colors you set are used in both modes.

All properties (colors are color ints, fonts are font resource IDs, text sizes are in sp, other sizes in dp; `0` for a color or font and `Float.NaN` for a size mean "use the SDK default"):

| Property | Default | Description |
|---|---|---|
| **General** | | |
| `backgroundColor` | light/dark default | Background of all screens and the status bar |
| `statusBarTextDark` | follows the system | Status bar text: `true` black, `false` white |
| `textColor` | light/dark default | All text: labels, texts, checkboxes, radio buttons, the back arrow |
| `textFont` | SDK default font | Font of all text that is not an input or a button |
| `viewTextSize` | 17 | Text size of texts and labels |
| `leadingSpace` | NaN | Space between the fields and the screen edges |
| `progressBarColor` | `#007AFF` | Circular progress shown while loading |
| `showBackButtonTitle` | `false` | Shows the BACK field's label next to the back arrow |
| **Inputs** | | |
| `inputFont` | `textFont` | Font of input fields |
| `inputTextSize` | 17 | Text size of input fields |
| `inputTextColor` | `textColor` | Text color of input fields |
| `inputBackgroundColor` | light/dark default | Background of input fields |
| `inputHeight` | NaN | Height of single-line input fields |
| `inputCornerRadius` | `buttonCornerRadius` | Corner radius of input fields |
| `inputBorderWidth` | NaN | Border width of input fields |
| `inputBorderColor` | `textColor` (focused: `buttonBackgroundColor`) | Border color of input fields |
| **Buttons** | | |
| `buttonFont` | system font | Font of buttons |
| `buttonTextSize` | 17 | Text size of buttons |
| `buttonTextColor` | black (white when no `FBAppearance` is passed) | Text color of buttons |
| `buttonBackgroundColor` | `#007AFF` | Background of buttons. Also the SDK's accent color: popups' buttons, the calendar popup, selected options |
| `buttonCornerRadius` | 8 | Corner radius of buttons |
| `buttonMinimumHeight` | NaN | Minimum height of buttons |
| `buttonSize` | `"medium"` | `"mini"`, `"small"`, `"medium"` or `"large"` |
| **Consent** | | |
| `consentTextSize` | 16 | Text size of the consent checkbox text |
| `consentLinkButtonFont` | the consent text's font | Font of the link in the consent text |
| **KYC** | | |
| `toastFont` | system font | Font of the directions text |
| `toastTextSize` | 17 | Text size of the directions text |
| `toastTextColor` | white | Color of the directions text |
| `embedInstructionsLabelInBubble` | `true` | Shows the directions in a bubble |
| `bubbleBackgroundColor` | default | Background of the bubble |
| `bubbleCornerRadius` | NaN (18 when a bubble color is set) | Corner radius of the bubble |
| `linearProgressIndicatorColor` | `#007AFF` | Progress indicator while scanning the ID |
| `linearProgressTrackColor` | default | Track of that progress indicator |
| `showCloseButton` | `false` | Cancel button on the KYC screen (white) |
| **Deprecated** | | |
| `viewTextColor`, `viewBackgroundColor` | — | Ignored: all text uses `textColor` and the screens use `backgroundColor` |

The `showCancelButton` intent extra is deprecated too: use `showCloseButton` instead.

### Calendar popup (DATE fields)

The header and the OK/Cancel buttons of the calendar popup use `buttonBackgroundColor`. Android can't change the color of the selected day at runtime, so it comes from the SDK's `FBDatePickerDialog` theme (default `#007AFF`). To use your own color, override that style by name in your app's `res/values/themes.xml` (or `styles.xml`). It changes only the SDK's calendar popup, nothing else in your app or the SDK:

```xml
<style name="FBDatePickerDialog" parent="Theme.MaterialComponents.DayNight.Dialog.Alert">
    <item name="colorControlActivated">@color/your_accent</item>
    <item name="android:colorControlActivated">@color/your_accent</item>
    <item name="colorAccent">@color/your_accent</item>
    <item name="android:colorAccent">@color/your_accent</item>
    <item name="colorPrimary">@color/your_accent</item>
    <item name="colorSecondary">@color/your_accent</item>
</style>
```

---

## Deeplinking (Android App Links for IBAN Verification)

If your verification flow includes the **IBAN Verification / Open Banking step**, users will be redirected to their banking app or mobile web banking to authenticate. To ensure the user returns to your application automatically once bank authorization is finished, you must configure **Android App Links**.

> **Note:** Contact the FormBuilder team to receive your assigned `{MERCHANT_SLUG}`.

### 1. Add Intent Filters to `AndroidManifest.xml`
In your app module's `AndroidManifest.xml`, merge the intent filters directly into `FormBuilderMainActivity`:

```xml
<activity
    android:name="com.formbuilder.sdk.presenter.activities.FormBuilderMainActivity"
    android:exported="true"
    tools:node="merge">

    <!-- FormBuilder Callback Redirect (Development) -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https" 
              android:host="dev3.kyc.rbbt.dev" 
              android:pathPrefix="/iris-redirect/{MERCHANT_SLUG}" />
    </intent-filter>

    <!-- FormBuilder Callback Redirect (Production) -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https" 
              android:host="id.rbbt.dev" 
              android:pathPrefix="/iris-redirect/{MERCHANT_SLUG}" />
    </intent-filter>
</activity>
```

### 2. Digital Asset Links Verification
For Android to verify the domain automatically without prompting the user with an "Open with..." dialog:
* Provide your app's **Package Name** (`applicationId`) and your **Release SHA-256 certificate fingerprint** to the FormBuilder team so they can add it to the server's `assetlinks.json`.
* Ensure `android:launchMode="singleTask"` or `singleTop` is set on the host activity so returning from the browser resumes the existing task instead of creating a duplicate instance.


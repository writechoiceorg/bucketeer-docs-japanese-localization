---
title: Flutter reference
slug: /sdk/client-side/flutter
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

This category contains topics explaining how to configure Bucketeer's Flutter SDK.

## Getting started

Before starting, ensure that you follow the [Getting Started](/getting-started) guide.

### Implementing dependency

Implement the dependency in your Pubspec file. Please refer to the [SDK releases page](https://github.com/bucketeer-io/flutter-client-sdk/releases) to find the latest version.

<Tabs>
<TabItem value="yaml" label="Pubspec">

```yaml showLineNumbers
dependencies:
  bucketeer_flutter_client_sdk: LATEST_VERSION
```

</TabItem>
</Tabs>

### Importing client

Import the Bucketeer client into your application code.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
import 'package:bucketeer_flutter_client_sdk/bucketeer_flutter_client_sdk.dart';
```

</TabItem>
</Tabs>

### Configuring client

Configure the SDK config and user configuration.

:::note

The **featureTag** setting is the tag you configure when creating a Feature Flag.

:::

All the settings in the example below are required.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
final config = BKTConfigBuilder()
  .apiKey("YOUR_API_KEY")
  .apiEndpoint("YOUR_API_URL")
  .featureTag("YOUR_FEATURE_TAG")
  .appVersion("YOUR_APP_VERSION")
  .build();

final user = BKTUserBuilder
  .id("USER_ID")
  .customAttributes(YOUR_USER_ATTRIBUTES) // optional Map<String, String>
  .build();
```

</TabItem>
</Tabs>

:::info Custom configuration

Depending on your use, you may want to change the optional configurations available in the **BKTConfigBuilder**.

- **pollingInterval** (Minimum 60 seconds. Default is 10 minutes)
- **backgroundPollingInterval** (Minimum 20 minutes. Default is 1 hour)
- **eventsFlushInterval** (Minimum 60 seconds. Default is 60 seconds)
- **eventsMaxQueueSize** (Default is 50 events)

:::

:::note

The Bucketeer SDK doesn't save the user data. The Application must save and set it when initializing the client SDK.

:::

### Initializing client

Initialize the client by passing the configurations in the previous step.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
await BKTClient.initialize(config: config, user: user);
final client = BKTClient.instance;
```

</TabItem>
</Tabs>

:::note

The initialize process immediately starts polling the latest evaluations from Bucketeer in the background using the interval `pollingInterval` configuration while the application is in the **foreground state**. When the application changes to the **background state**, it will use the `backgroundPollingInterval` configuration.

:::

If you want to use the feature flag on Splash or Main views, and the user opens your application for the first time, it may not have enough time to fetch the variations from the Bucketeer server.

For this case, we recommend using the `timeout` option in the initialize method. It will block until the SDK receives the latest user variations.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
/// It will unlock without waiting until the fetching variation process finishes
int timeout = 1000;

await BKTClient.initialize(config: config, user: user, timeoutMillis: timeout);
final client = BKTClient.instance;
```

</TabItem>
</Tabs>

## Supported features

### Evaluating user

The variation method determines whether or not a feature flag is enabled for a specific user.<br />
To check which variation a specific user will receive, you can use the client like below.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
final showNewFeature = await client.boolVariation("YOUR_FEATURE_FLAG_ID", false);
if (showNewFeature) {
    /// The Application code to show the new feature
} else {
    /// The code to run if the feature is off
}
```

</TabItem>
</Tabs>

:::caution

In case of the feature flag is missing in the SDK, it will return the default value.

:::

### Variation types

The Bucketeer SDK supports the following variation types.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
Future<bool> boolVariation(String featureId, { required bool defaultValue });

Future<String> stringVariation(String featureId, { required String defaultValue });

Future<int> intVariation(String featureId, { required int defaultValue });

Future<double> doubleVariation(String featureId, { required double defaultValue });

Future<Map<String, dynamic>> jsonVariation(String featureId, { required Map<String, dynamic> defaultValue });
```

</TabItem>
</Tabs>

### Updating user evaluations

Depending on the use case, you may need to ensure the evaluations in the SDK are up to date before requesting the variation.

The fetch method uses the following parameter. Make sure to wait for its completion.

- **Timeout** (Default is 30 seconds)

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
/// It will unlock without waiting until the fetching variation process finishes
int timeout = 5000;

final result = await client.fetchEvaluations(timeoutMillis: timeout);
if (result.isSuccess) {
  final showNewFeature = await client.boolVariation("YOUR_FEATURE_FLAG_ID", defaultValue: false);
  if (showNewFeature) {
      /// The Application code to show the new feature
  } else {
      /// The code to run if the feature is off
  }
} else {
   /// The code to run if the feature is off
}
```

</TabItem>
</Tabs>

:::caution

Depending on the client network, it could take a few seconds until the SDK fetches the data from the server, so use this carefully.

You don't need to call this method manually in regular use because the SDK is polling the latest variations in the background.

:::

### Updating user evaluations in real-time

The Bucketeer SDK supports FCM ([Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)).<br />
Every time you change a feature flag on the admin console, Bucketeer will send notifications using the FCM API to notify the client so that you can update the evaluations in real-time.

:::note

1. You need to register your FCM API Key on the admin console. Check the [Pushes](/integration/pushes) section to learn how to do it.
2. This feature may not work if the user has the notification disabled.

:::

Assuming you already have the FCM implementation in your application.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
FirebaseMessaging.onMessage.listen((RemoteMessage message) async {
  final isFeatureFlagUpdated = message.data["bucketeer_feature_flag_updated"]
    if (isFeatureFlagUpdated) {
      int timeout = 1000;

      final result = await client.fetchEvaluations(timeoutMillis: timeout);
      if (result.isSuccess) {
        final showNewFeature = await client.boolVariation("YOUR_FEATURE_FLAG_ID", defaultValue: false);
        if (showNewFeature) {
            /// The Application code to show the new feature
        } else {
            /// The code to run if the feature is off
        }
      } else {
        /// The code to run if the feature is off
      }
    }
});
```

</TabItem>
</Tabs>

### Reporting custom events

This method lets you save user actions in your application as events. You can connect these events to metrics in the experiments console UI.

In addition, you can pass a double value to the goal event. These values will sum and show as <br />`Value total` on the experiments console UI. This is useful if you have a goal event for tracking how much a user spent on your application buying items, etc.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
await client.track("YOUR_GOAL_ID", value: 10.50);
```

</TabItem>
</Tabs>

### Flushing events

This method will send all pending analytics events to the Bucketeer server immediately. This process is asynchronous, so it returns before it is complete.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
client.flush();
```

</TabItem>
</Tabs>

:::note

In regular use, you don't need to call the flush method because the events are sent every 30 seconds in the background.

:::

### User attributes configuration

This feature will give you robust and granular control over what users can see on your application. You can add rules using these attributes on the console UI's feature flag's targeting tab. [See more](/feature-flags/creating-feature-flags/targeting#user-attributes).

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
final attributes = {
  'app_version': '1.0.0',
  'os_version': '11.0.0',
  'device_model': 'pixel-5'
  'language': 'english',
  'genre': 'female'
};

final user = BKTUserBuilder()
  .id("USER_ID")
  .customAttributes(attributes)
  .build();

await BKTClient.initialize(config: config, user: user);
```

</TabItem>
</Tabs>

### Updating user attributes

This method will update all the current user attributes. This is useful in case the user attributes update dynamically on the application after initializing the SDK.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
final attributes = {
  'app_version': '1.0.1',
  'os_version': '11.0.0',
  'device_model': 'pixel-5'
  'language': 'english',
  'genre': 'female'
  'country': 'japan'
};

await client.updateUserAttributes(attributes);
```

</TabItem>
</Tabs>

:::caution

This updating method will override the current data.

:::

### Getting user information

This method will return the current user configured in the SDK. This is useful when you want to check the current user id and attributes before updating them through [updateUserAttributes](flutter#updating-user-attributes).

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
final user = await client.currentUser();
```

</TabItem>
</Tabs>

### Getting evaluation details

This method will return the evaluation details for a specific feature flag. This is useful if you need to know the variation reason or send this data elsewhere.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
final evaluationDetails = await client.evaluationDetails("YOUR_FEATURE_FLAG_ID");
```

:::note

This method will return null if the feature flag is missing in the SDK.

:::

</TabItem>
</Tabs>


### Listening to evaluation updates

BKTClient can notify when the evaluation is updated.
The listener can detect both automatic polling and manual fetching.

<Tabs>
<TabItem value="dart" label="Dart">

```dart showLineNumbers
class EvaluationUpdateListenerImpl implements BKTEvaluationUpdateListener {
  @override
  void onUpdate() {
    final client = BKTClient.instance;
    final showNewFeature = client.boolVariation("YOUR_FEATURE_FLAG_ID", defaultValue: false);
    if (showNewFeature) {
      /// The Application code to show the new feature
    } else {
      /// The code to run when the feature is off
    }
  }
}

final client = BKTClient.instance;

final listener = EvaluationUpdateListenerImpl();
/// The returned key value is used to remove the listener
final key = client.addEvaluationUpdateListener(listener);

/// Remove a listener associated with the key
client.removeEvaluationUpdateListener(key: key)

/// Remove all listeners
client.clearEvaluationUpdateListeners()
```

</TabItem>
</Tabs>

### Background mode

For Android, the background mode works without any configuration by default.<br />
For iOS, this feature is optional, but it can be used by configuring the background mode in the Xcode project.<br />
The default setting is **1 hour** and a minimum of **20 minutes** for the `backgroundPollingInterval`, and **1 minute** for the `eventsFlushInterval` settings.

#### Configuration for iOS
**1.** Open the iOS folder using Xcode IDE to open the file `Runner.xcworkspace`.<br />
**2.** Open the XCode project setting.<br />
**3.** Select the `Runner` target.<br />
**4.** Select the `Signing & Capabilities` settings tab.<br />
**5.** Enable the `Background processing` option in the `Background Modes` section.<br />
**6.** Register the identifier by adding the `Permitted background task scheduler identifiers` option in the **Info.plist**, and set the following task IDs as the values.

-  **io.bucketeer.background.fetch.evaluations** (Background task to fetch the latest evaluations from the server).
-  **io.bucketeer.background.flush.events** (Background task to flush events generated by the client).

**7.** Open the `AppDelegate.swift` and add the following code to the `didFinishLaunchingWithOptions` function.

```swift showLineNumbers
import UIKit
import Bucketeer

class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Add the code to enable background tasks
        if #available(iOS 13.0, tvOS 13.0, *) {
            BKTBackgroundTask.enable()
        }
        // Your app logic code
        return true
    }
```

**8.** If needed, change the background polling default interval by adding the `backgroundPollingInterval` setting in the **BKTConfigBuilder** as follows:

```dart showLineNumbers
final config = BKTConfigBuilder()
  .apiKey("YOUR_API_KEY")
  .apiEndpoint("YOUR_API_URL")
  .featureTag("YOUR_FEATURE_TAG")
  .appVersion("YOUR_APP_VERSION").
  .eventsFlushInterval(60) // 1 minute
  .backgroundPollingInterval(1800)
  .build(); // 30 minutes
```

**9.** Please check the [iOS Documentation](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background/using_background_tasks_to_update_your_app) for more details.

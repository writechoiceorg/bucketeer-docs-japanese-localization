---
title: Android reference
slug: /sdk/client-side/android
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

This category contains topics explaining how to configure Bucketeer's Android SDK.

:::info Compatibility

The Bucketeer SDK is compatible with Android SDK versions 21 and higher (Android 5.0, Lollipop).

:::

## Getting started

Before starting, ensure that you follow the [Getting Started](/getting-started) guide.

### Implementing dependency

Implement the dependency in your Gradle file. Please refer to the [SDK releases page](https://github.com/bucketeer-io/android-client-sdk/releases) to find the latest version.

<Tabs>
<TabItem value="gradle" label="Gradle">

```groovy showLineNumbers
dependencies {
  implementation 'io.bucketeer:android-client-sdk:LATEST_VERSION'
}
```

</TabItem>
</Tabs>

### Importing client

Import the Bucketeer client into your application code.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
import io.bucketeer.sdk.android.*
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
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val config = BKTConfig.builder()
  .apiKey("YOUR_API_KEY")
  .apiEndpoint("YOUR_API_ENDPOINT")
  .featureTag("YOUR_FEATURE_TAG")
  .build()

val user = BKTUser.builder()
  .id("USER_ID")
  .build()
```

</TabItem>
</Tabs>

:::info Custom configuration

Depending on your use, you may want to change the optional configurations available in the **BKTConfig.Builder**.

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
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
BKTClient.initialize(this.application, config, user)
val client = BKTClient.getInstance()
```

</TabItem>
</Tabs>

:::note

The initialize process immediately starts polling the latest evaluations from Bucketeer in the background using the interval `pollingInterval` configuration while the application is in the **foreground state**.
When the application changes to the **background state**, it will use the `backgroundPollingInterval` configuration.

:::

If you want to use the feature flag on Splash or Main views, and the user opens your application for the first time, it may not have enough time to fetch the variations from the Bucketeer server.

For this case, we recommend using the `Future<BKTException?>` returned from the initialize method.

:::note

Be aware that if an error is returned in the initialize process, it is regarding the fetch evaluation timeout error, not the initialize process itself.

:::

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val timeout = 1000 // Default is 5 seconds

viewLifecycleOwner.lifecycleScope.launch {
  val future = BKTClient.initialize(this.application, config, user, timeout)

  // Future is blocking, so you need to wait on non-main thread.
  val error = withContext(Dispatchers.IO) {
    future.get()
  }

  // Future returns null if BKTClient successfully fetched evaluations.
  if (error == null) {
    val showNewFeature = BKTClient.getInstance().booleanVariation("YOUR_FEATURE_FLAG_ID", false)
    if (showNewFeature) {
        // The Application code to show the new feature
    } else {
        // The code to run when the feature is off
    }
  } else {
    // Handle the error when there is no cache or the cache is not updated
  }
}
```

</TabItem>
</Tabs>

## Supported features

### Evaluating user

The variation method determines whether or not a feature flag is enabled for a specific user.<br />
You can use the client like the one below to check which variation a specific user will receive.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val showNewFeature = client.booleanVariation("YOUR_FEATURE_FLAG_ID", false)
if (showNewFeature) {
    // The Application code to show the new feature
} else {
    // The code to run when the feature is off
}
```

:::note

The variation method will return the default value if the feature flag is missing in the SDK.

:::

</TabItem>
</Tabs>

### Variation types

The Bucketeer SDK supports the following variation types.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
fun booleanVariation(featureId: String, defaultValue: Boolean): Boolean

fun stringVariation(featureId: String, defaultValue: String): String

fun intVariation(featureId: String, defaultValue: Int): Int

fun doubleVariation(featureId: String, defaultValue: Double): Double

fun jsonVariation(featureId: String, defaultValue: JSONObject): JSONObject
```

</TabItem>
</Tabs>

### Updating user evaluations

Depending on the use case, you may need to ensure the evaluations in the SDK are up to date before requesting the variation.

The fetch method uses the following parameter and returns a `Future<BKTExeptIon?>`.

- **Timeout** (Default is 30 seconds)

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val timeout = 5000
val future = client.fetchEvaluations(timeout)

// Future is blocking, avoid waiting it on the main thread.
val error = future.get()
if (error == null) {
  val showNewFeature = client.booleanVariation("YOUR_FEATURE_FLAG_ID", false)
  if (showNewFeature) {
      // The Application code to show the new feature
  } else {
      // The code to run when the feature is off
  }
} else {
  // Handle the error
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
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
override fun onMessageReceived(remoteMessage: RemoteMessage?) {
  remoteMessage?.data?.also { data ->
    val isFeatureFlagUpdated = data["bucketeer_feature_flag_updated"]
    if (isFeatureFlagUpdated) {
      // The callback will return without waiting until the fetching variation process finishes
      val timeout = 1000 // Default is 5 seconds

      val future = client.fetchEvaluations(timeout)
      val error = future.get()
      if (error == null) {
        val showNewFeature = client.booleanVariation("YOUR_FEATURE_FLAG_ID", false)
        if (showNewFeature) {
            // The Application code to show the new feature
        } else {
            // The code to run when the feature is off
        }
      } else {
        // Handle the error
      }
    }
  }
}
```

</TabItem>
</Tabs>

### Reporting custom events

This method lets you save user actions in your application as events. You can connect these events to metrics in the experiments console UI.

In addition, you can pass a double value to the goal event. These values will sum and show as <br />`Value total` on the experiments console UI. This is useful if you have a goal event for tracking how much a user spent on your application buying items, etc.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
client.track("YOUR_GOAL_ID", 10.50)
```

</TabItem>
</Tabs>

### Flushing events

This method will send all pending analytics events to the Bucketeer server immediately. This process is asynchronous, but the method returns `Future<BKTExeptIon?>` if you want to wait for its completion before doing something else.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val future = client.flush()
```

</TabItem>
</Tabs>

:::note

In regular use, you don't need to call the flush method because the events are sent every 30 seconds in the background.

:::

### User attributes configuration

This feature will give you robust and granular control over what users can see on your application. You can add rules using these attributes on the console UI's feature flag's targeting tab. [See more](/feature-flags/creating-feature-flags/targeting#user-attributes).

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val attributes = mapOf(
  "app_version" to "1.0.0",
  "os_version" to "11.0.0",
  "device_model" to "pixel-5"
  "language" to "english",
  "genre" to "female",
)

val user = BKTUser.builder()
  .id("USER_ID")
  .customAttributes(attributes)
  .build()

BKTClient.initialize(this.application, config, user)
val client = BKTClient.getInstance()
```

</TabItem>
</Tabs>

### Updating user attributes

This method will update all the current user attributes. This is useful in case the user attributes update dynamically on the application after initializing the SDK.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val attributes = mapOf(
  "app_version" to "1.0.1",
  "os_version" to "11.0.0",
  "device_model" to "pixel-5"
  "language" to "english",
  "genre" to "female",
  "country" to "japan",
)

client.updateUserAttributes(attributes)
```

</TabItem>
</Tabs>

:::caution

This updating method will override the current data.

:::

### Getting user information

This method will return the current user configured in the SDK. This is useful when you want to check the current user id and attributes before updating them through [updateUserAttributes](android#updating-user-attributes).

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val user = client.currentUser()
```

</TabItem>
</Tabs>

### Getting evaluation details

This method will return the evaluation details for a specific feature flag. It is useful if you need to know the variation reason or send this data elsewhere.
It will return null if the feature flag is missing in the SDK.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
val evaluationDetails = client.evaluationDetails("YOUR_FEATURE_FLAG_ID")
```

:::caution

Do not call this method without calling the [Evaluating user method](android#evaluating-user). The Evaluating user method must always be called because it generates analytics events that will be sent to the server.

:::

</TabItem>
</Tabs>


### Listening to evaluation updates

BKTClient can notify when the evaluation is updated.  
The listener can detect both automatic polling and manual fetching.

<Tabs>
<TabItem value="kt" label="Kotlin">

```kotlin showLineNumbers
// Returned value is used when you want to remove listener
val key = client.addEvaluationUpdateListener {
  val showNewFeature = client.booleanVariation("YOUR_FEATURE_FLAG_ID", false)
  if (showNewFeature) {
      // The Application code to show the new feature
  } else {
      // The code to run when the feature is off
  }
}

// Remove a listener associated with the key
client.removeEvaluationUpdateListener(key)

// Remove all listeners
client.clearEvaluationUpdateListeners()
```

</TabItem>
</Tabs>

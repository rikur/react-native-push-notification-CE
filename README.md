# React Native Push Notifications Community Edition

This Library was cloned from the original @Zo0r/react-native-push-notification. Under the MIT license and for the sake of making sure this library continues to be supported, I cloned it and created a new repo. If you'd like to support/contribute this library, please reach out to me and I will make it happen.

## Thank you Zo0r for all of your hard work in creating this awesome library

## Installation
`npm install --save react-native-push-notification-ce` or `yarn add react-native-push-notification-ce`

`react-native link`

**NOTE: For Android, you will still have to manually update the AndroidManifest.xml (as below) in order to use Scheduled Notifications.**

## Issues

Having a problem?  Read the [troubleshooting](./trouble-shooting.md) guide before raising an issue.

## Pull Requests

[Please read...](./submitting-a-pull-request.md)

## iOS manual Installation
The component uses PushNotificationIOS for the iOS part.

[Please see: PushNotificationIOS](https://facebook.github.io/react-native/docs/pushnotificationios.html#content)

## Android manual Installation

**NOTE: To use a specific `play-service-gcm` or `firebase-messaging` version:**

In your `android/build.gradle`
```gradle
ext {
    googlePlayServicesVersion = "<Your play services version>" // default: "+"
    firebaseVersion = "<Your Firebase version>" // default: "+"
    // Other settings
    compileSdkVersion = "<Your compile SDK version>" // default: 23
    buildToolsVersion = "<Your build tools version>" // default: "23.0.1"
    targetSdkVersion = "<Your target SDK version>" // default: 23
    supportLibVersion = "<Your support lib version>" // default: 23.1.1
}
```

In your `AndroidManifest.xml`
```xml
    .....
    <!-- <Only if you're using GCM> -->
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <permission
        android:name="${applicationId}.permission.C2D_MESSAGE"
        android:protectionLevel="signature" />
    <uses-permission android:name="${applicationId}.permission.C2D_MESSAGE" />
    <!-- </Only if you're using GCM> -->


    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

    <application ....>
    <!-- <Only if you're using GCM> -->
        <receiver
            android:name="com.google.android.gms.gcm.GcmReceiver"
            android:exported="true"
            android:permission="com.google.android.c2dm.permission.SEND" >
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="${applicationId}" />
            </intent-filter>
        </receiver>
    <!-- </Only if you're using GCM> -->
        <receiver android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationPublisher" />
        <receiver android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationBootEventReceiver">
            <intent-filter>
            <!-- <Only if you're using GCM> -->
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            <!-- </Only if you're using GCM> -->
            <!-- <Else> -->
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            <!-- </Else> -->
            </intent-filter>
        </receiver>
        <service android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationRegistrationService"/>
        <service
            android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationListenerService"
            android:exported="false" >
            <intent-filter>
            <!-- <Only if you're using GCM> -->
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <!-- </Only if you're using GCM> -->
            <!-- <Else> -->
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            <!-- </Else> -->
            </intent-filter>
        </service>
     .....

```

In `android/settings.gradle`
```gradle
...

include ':react-native-push-notification-ce'
project(':react-native-push-notification-ce').projectDir = file('../node_modules/react-native-push-notification-ce/android')
```

Manually register module in `MainApplication.java` (if you did not use `react-native link`):

```java
import com.dieam.reactnativepushnotification.ReactNativePushNotificationPackage;  // <--- Import Package

public class MainApplication extends Application implements ReactApplication {

  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
      @Override
      protected boolean getUseDeveloperSupport() {
        return BuildConfig.DEBUG;
      }

      @Override
      protected List<ReactPackage> getPackages() {

      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
          new ReactNativePushNotificationPackage() // <---- Add the Package
      );
    }
  };

  ....
}
```

## Usage
```javascript
var PushNotification = require('react-native-push-notification-ce');

PushNotification.configure({

    // (optional) Called when Token is generated (iOS and Android)
    onRegister: function(token) {
        console.log( 'TOKEN:', token );
    },

    // (required) Called when a remote or local notification is opened or received
    onNotification: function(notification) {
        console.log( 'NOTIFICATION:', notification );

        // process the notification

        // required on iOS only (see fetchCompletionHandler docs: https://facebook.github.io/react-native/docs/pushnotificationios.html)
        notification.finish(PushNotificationIOS.FetchResult.NoData);
    },

    // ANDROID ONLY: GCM or FCM Sender ID (product_number) (optional - not required for local notifications, but is need to receive remote push notifications)
    senderID: "YOUR GCM (OR FCM) SENDER ID",

    // IOS ONLY (optional): default: all - Permissions to register.
    permissions: {
        alert: true,
        badge: true,
        sound: true
    },

    // Should the initial notification be popped when appStart() gets called
    // default: true
    popInitialNotification: true,

    /**
      * (optional) default: true
      * - Specified if permissions (ios) and token (android and ios) will requested or not,
      * - if not, you must call PushNotificationsHandler.requestPermissions() later
      */
    requestPermissions: true,
});
```

On your root react component in the componentDidMount lifecycle make sure to call:

```
PushNotification.appStart() // Tells the bridge when the app has started so it can display tapped notifications
```

## Handling Notifications
When any notification is opened or received the callback `onNotification` is called passing an object with the notification data.

Notification object example:
```javascript
{
    foreground: false, // BOOLEAN: If the notification was received in foreground or not
    userInteraction: false, // BOOLEAN: If the notification was opened by the user from the notification area or not
    message: 'My Notification Message', // STRING: The notification message
    data: {}, // OBJECT: The push data
}
```

## Local Notifications
`PushNotification.localNotification(details: Object)`

EXAMPLE:
```javascript
PushNotification.localNotification({
    /* Android Only Properties */
    id: '0', // (optional) Valid unique 32 bit integer specified as string. default: Autogenerated Unique ID
    ticker: "My Notification Ticker", // (optional)
    autoCancel: true, // (optional) default: true
    largeIcon: "ic_launcher", // (optional) default: "ic_launcher"
    smallIcon: "ic_notification", // (optional) default: "ic_notification" with fallback for "ic_launcher"
    bigText: "My big text that will be shown when notification is expanded", // (optional) default: "message" prop
    subText: "This is a subText", // (optional) default: none
    color: "red", // (optional) default: system default
    vibrate: true, // (optional) default: true
    vibration: 300, // vibration length in milliseconds, ignored if vibrate=false, default: 1000
    tag: 'some_tag', // (optional) add tag to message
    group: "group", // (optional) add group to message
    ongoing: false, // (optional) set whether this is an "ongoing" notification

    /* iOS only properties */
    alertAction: // (optional) default: view
    category: // (optional) default: null
    userInfo: // (optional) default: null (object containing additional notification data)

    /* iOS and Android properties */
    title: "My Notification Title", // (optional)
    message: "My Notification Message", // (required)
    playSound: false, // (optional) default: true
    soundName: 'default', // (optional) Sound to play when the notification is shown. Value of 'default' plays the default sound. It can be set to a custom sound such as 'android.resource://com.xyz/raw/my_sound'. It will look for the 'my_sound' audio file in 'res/raw' directory and play it. default: 'default' (default sound is played)
    number: '10', // (optional) Valid 32 bit integer specified as string. default: none (Cannot be zero)
    repeatType: 'day', // (Android only) Repeating interval. Could be one of `week`, `day`, `hour`, `minute, `time`. If specified as time, it should be accompanied by one more parameter 'repeatTime` which should the number of milliseconds between each interval
    actions: '["Yes", "No"]',  // (Android only) See the doc for notification actions to know more
});
```

## Scheduled Notifications
`PushNotification.localNotificationSchedule(details: Object)`

EXAMPLE:
```javascript
PushNotification.localNotificationSchedule({
  message: "My Notification Message", // (required)
  date: new Date(Date.now() + (60 * 1000)) // in 60 secs
});
```

## Custom sounds

In android, add your custom sound file to `[project_root]/android/app/src/main/res/raw`

In iOS, add your custom sound file to the project `Resources` in xCode.

In the location notification json specify the full file name:

    soundName: 'my_sound.mp3'

## Cancelling notifications

### 1) cancelLocalNotifications

`PushNotification.cancelLocalNotifications(details);`

The the `details` parameter allows you to specify a `userInfo` dictionary that can be used to match one or more *scheduled* notifications.  Each
matched notification is cancelled and its alerts removed from the notification centre.  The RN docs suggest this is an optional parameter, but
it is not.

```javascript
PushNotification.cancelLocalNotifications({id: '123'});
```

### 2) cancelAllLocalNotifications

`PushNotification.cancelAllLocalNotifications()`

Cancels all scheduled notifications AND clears the notifications alerts that are in the notification centre.

*NOTE: there is currently no api for removing specific notification alerts from the notification centre.*

## Repeating Notifications ##

(Android only) Specify `repeatType` and optionally `repeatTime` while scheduling the local notification. Check the local notification example above.

For iOS, the repeating notification should land soon. It has already been merged to the [master](https://github.com/facebook/react-native/pull/10337)

## Notification Actions ##

(Android only) [Refer](https://github.com/zo0r/react-native-push-notification/issues/151) to this issue to see an example of a notification action.

Two things are required to setup notification actions.

### 1) Specify notification actions for a notification
This is done by specifying an `actions` parameters while configuring the local notification. This is an array of strings where each string is a notification action that will be presented with the notification.

For e.g. `actions: '["Accept", "Reject"]'  // Must be in string format`

The array itself is specified in string format to circumvent some problems because of the way JSON arrays are handled by react-native android bridge.

### 2) Specify handlers for the notification actions
For each action specified in the `actions` field, we need to add a handler that is called when the user clicks on the action. This can be done in the `componentWillMount` of your main app file or in a separate file which is imported in your main app file. Notification actions handlers can be configured as below:

```
import PushNotificationAndroid from 'react-native-push-notification-ce'

(function() {
  // Register all the valid actions for notifications here and add the action handler for each action
  PushNotificationAndroid.registerNotificationActions(['Accept','Reject','Yes','No']);
  DeviceEventEmitter.addListener('notificationActionReceived', function(action){
    console.log ('Notification action received: ' + action);
    const info = JSON.parse(action.dataJSON);
    if (info.action == 'Accept') {
      // Do work pertaining to Accept action here
    } else if (info.action == 'Reject') {
      // Do work pertaining to Reject action here
    }
    // Add all the required actions handlers
  });
})();
```

For iOS, you can use this [package](https://github.com/holmesal/react-native-ios-notification-actions) to add notification actions.

## Set application badge icon

`PushNotification.setApplicationIconBadgeNumber(number: number)`

Works natively in iOS.

Uses the [ShortcutBadger](https://github.com/leolin310148/ShortcutBadger) on Android, and as such will not work on all Android devices.

## Sending Notification Data From Server
Same parameters as `PushNotification.localNotification()`

## Android Only Methods

`PushNotification.subscribeToTopic(topic: string)` Subscribe to a topic (works only with Firebase)

## Checking Notification Permissions
`PushNotification.checkPermissions(callback: Function)` Check permissions
`callback` will be invoked with a `permissions` object:
 `alert`: boolean
 `badge`: boolean
 `sound`: boolean

## iOS Only Methods
`PushNotification.checkPermissions(callback: Function)` Check permissions

`PushNotification.getApplicationIconBadgeNumber(callback: Function)` get badge number

`PushNotification.abandonPermissions()` Abandon permissions

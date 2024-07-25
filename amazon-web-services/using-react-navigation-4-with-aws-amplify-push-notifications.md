# Using React Navigation 4 with AWS Amplify Push Notifications
**Daniel Dantas** - **Jan. 28, 2020**

> **Since this post was written, React Navigation now has an** [**easier method**](https://reactnavigation.org/docs/navigating-without-navigation-prop#handling-initialization) **of navigating before the navigator finishes mounting. I’m leaving this post up for historical reasons.**

This post describes one method to trigger React Navigation 4 navigation from push notifications using AWS Amplify.  
[An alternate method is to use deep linking](https://reactnavigation.org/docs/en/4.x/deep-linking.html), but there are certain use cases that I encountered where the method in this post — using AWS Amplify push notification handler functions to navigate — is preferred.  
Use whichever method — deep linking, or push notification handler functions — is better for your situation.

The example code for this post uses React Native 61.5, React Navigation 4.0.10, and AWS Amplify 2.2.2, and is at [https://github.com/dantasfiles/AmplifyAndroidPush](https://github.com/dantasfiles/AmplifyAndroidPush/tree/react-navigation-4)

I have tested this with Android, but have not yet tested this with iOS, so the push notification configuration functions (`PushNotification.onNotification` and `PushNotification.onNotificationOpened`)  may behave somewhat differently with iOS. Until I can do more testing and update this post, you’ll have to adapt based on your experiences.

## Install React Navigation 4

**Install React Navigation 4**: [**Getting Started**](https://reactnavigation.org/docs/4.x/getting-started) → [**Installing Stack Navigator**](https://reactnavigation.org/docs/4.x/stack-navigator.html)

## Create the Navigation Structure

In our example, we use a **full-screen modal** to display the notification, based on [**Opening a Full-Screen Modal**](https://reactnavigation.org/docs/en/4.x/modal.html) in the React Navigation 4 documentation.

**[App.js](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/App.js#L29)**
```js
import {createAppContainer} from 'react-navigation';  
import {createStackNavigator} from 'react-navigation-stack';  
const MainStack = createStackNavigator(  
  { Home: { screen: HomeScreen, }, },  
  {},  
);  
const RootStack = createStackNavigator(  
  {  
    Main: { screen: MainStack, },  
    Notification: { screen: NotificationScreen, },  
  },  
  { mode: 'modal', headerMode: 'none', },  
);  
const AppContainer = createAppContainer(RootStack);
```

The main navigation structure of your app is in **`MainStack`**. Right now it only contains the [**`HomeScreen`**](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/HomeScreen.js), but you can fill this in with the rest of your application navigation structure.  
The **`RootStack`**  sits on top of the **`MainStack`**, and displays either the app’s normal **`MainStack`**  navigation structure,  or the [**`NotificationScreen`**](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/NotificationScreen.js), which is displayed when a notification arrives or when the user opens the app from a notification.

## Discussion of push notification configuration functions

There are two AWS Amplify push notification configuration functions that specify how to handle incoming push notifications: **`PushNotification.onNotification`** and **`PushNotification.onNotificationOpened`**

**[App.js](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/App.js#L14)**
```js
PushNotification.onNotification(notification => {  
  if (notification.foreground) {   
    // NOTIFICATION RECEIVED WHEN APP IS IN FOREGROUND  
    ...  
  }  
  else {   
    // NOTIFICATION RECEIVED WHEN APP IN BACKGROUND OR CLOSED  
    ...   
  }  
  ...  
  // iOS only  
  notification.finish(PushNotificationIOS.FetchResult.NoData);  
});  
PushNotification.onNotificationOpened(notification => {  
  // USER CLICKED ON PUSH NOTIFICATION  
  ...  
});
```

The details of those configuration functions are explored in [**Handling Incoming Push Notifications in AWS Amplify**](handling-incoming-push-notifications-in-aws-amplify.md). The behavior of your handler functions will be different depending on whether the app is in the foreground, background, or closed, and whether the user clicks the push notification or just opens the app. **Before you decide on your navigation logic and which handler function to put it in, you should read that post carefully to determine how your navigation logic will interact with push notification behavior.**

In the example in this post, we’ll navigate when the [`PushNotification.onNotification`](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/App.js#L14) handler function is triggered. This means that the notification screen will always be shown after a push notification arrives. If the app is in the foreground, the navigation to the notification screen will be instant. If the app is in the background or closed, the navigation to the notification screen will occur when the app is opened, regardless of whether the user manually opens the app or clicks the push notification.

If you choose instead to place your navigation code in your [`PushNotification.onNotificationOpened`](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/App.js#L22) handler function, then the app will only navigate to the notification screen when the user clicks a push notification when the app is in the background or closed. If the app is in the foreground, or if the background or closed app is opened by clicking its icon — not by clicking the push notification — your `PushNotification.onNotificationOpened` handler function will not be triggered and no navigation will occur.

Finally, you can include navigation logic in both the `PushNotification.onNotification` and `PushNotification.onNotificationOpened` handler functions to create complex navigation patterns.

## Extract the data from the push notification

Recall from [**Testing Push Notifications with AWS Amplify**](testing-push-notifications-with-aws-amplify.md) that you can use the AWS CLI to [send a push notification](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/pinpoint-send-messages.json). We will use the `data` field in [that push notification](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/pinpoint-send-messages.json) to specify the **screen to navigate to** (in our example, we only have a single `NotificationScreen` but your app can have multiple navigation possibilities), and **some data to display on that screen**:

```js
{ ...  
    "MessageConfiguration": {  
      "DefaultPushNotificationMessage": {  
        "Body": "Test Body",  
        "Title": "Test Title",  
        "Data": {  
          "screen": "Notification",  
          "Data Field 1": "Data Value 1",  
          "Data Field 2": "Data Value 2"  
        }  
      }  
    }  
  }  
}
```

When the above push notification is received, the `notification` object passed to the `PushNotification.onNotification` handler function  will have the following form:

```js
{  
  body: PUSH NOTIFICATION BODY,  
  data: {  
    "pinpoint.campaign.campaign_id": "_DIRECT",  
    "pinpoint.jsonBody": "{\"Data Field 1\":\"Data Value 1\",\"Data Field 2\":\"Data Value 2\",\"screen\":\"Notification\"}"  
    "pinpoint.notification.body": PUSH NOTIFICATION BODY,  
    "pinpoint.notification.silentPush": "0",  
    "pinpoint.notification.title": PUSH NOTIFICATION TITLE,  
    "pinpoint.openApp": "true"  
    ...  
  },  
  foreground: IS APP IN FOREGROUND OR BACKGROUND,  
  title: PUSH NOTIFICATION TITLE  
}
```

We can parse that JSON string to extract our data.

**[App.js](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/App.js#L14)**  
```js 
PushNotification.onNotification(notification => {  
  const data = JSON.parse(notification.data\['pinpoint.jsonBody'\]);  
  const screen = data.screen;  
  ...  
});
```

## Navigating without the navigation prop

The push notification configuration function `PushNotification.onNotification` **must be run outside of a React component**. If it is run inside a React component, then if the app is closed and opened by a push notification, the `PushNotification.onNotification` configuration function will not yet have run when the push notification is handled, and your `PushNotification.onNotification` handler function will not be triggered.

What that means is that we can’t use the [normal method of navigation inside React components](https://reactnavigation.org/docs/en/4.x/navigating.html) using **`props.navigation.navigate()`**. Instead, we use a modified form of the technique from [**Navigating without the navigation prop**](https://reactnavigation.org/docs/en/4.x/navigating-without-navigation-prop.html).

**[App.js](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/App.js#L54)**
```js
const AppContainer = createAppContainer(RootStack);  
const App = withAuthenticator(() => {  
  return (  
    <AppContainer  
      ref ={navigatorRef => {  
        NavigationService.setTopLevelNavigator(navigatorRef);  
      }}  
    />  
  );  
});  
export default App;
```

In the above code, we get access to a navigator through a `ref` and pass it to the [`NavigationService`](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/NavigationService.js) (defined later) which we will use to navigate.

> Note: If you modify this example code, make sure to only call HOC’s like [`withAuthenticator`](https://aws-amplify.github.io/docs/js/authentication#using-withauthenticator-hoc) and [`createAppContainer`](https://reactnavigation.org/docs/en/4.x/hello-react-navigation.html#creating-a-stack-navigator) at the top-level, not inside a React function component. See [Don’t Use HOCs Inside the render Method](https://reactjs.org/docs/higher-order-components.html#dont-use-hocs-inside-the-render-method).

**[App.js](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/App.js#L14)**
```js
PushNotification.onNotification(notification => {  
  const data = JSON.parse(notification.data\['pinpoint.jsonBody'\]);  
  const screen = data.screen;  
  NavigationService.navigate(screen, {data});  
  // iOS only  
  notification.finish(PushNotificationIOS.FetchResult.NoData);  
});
```

In the above code, we use our [`NavigationService`](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/NavigationService.js) (defined later) to navigate to the `screen` specified by our push notification, and pass that screen the `data` passed from our push notification.

**[NotificationScreen.js](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/NotificationScreen.js)**
```js 
function NotificationScreen(props) {  
  const data = props.navigation.state.params.data;  
  ...  
}
```

In the above code, we extract the `data` passed from our push notification in order to use or display it.

## NavigationService

We can’t use the default implementation of `NavigationService` in [**Navigating without the navigation prop**](https://reactnavigation.org/docs/en/4.x/navigating-without-navigation-prop.html), because in the case where the app is closed and the user clicks a push notification to open the app, the app will break. The push notification handler function will run and will try to navigate with **`NavigationService.navigate()`**  before the React components are rendered and **`NavigationService.setTopLevelNavigator()`** is set.

So we use a modification of the techniques from [“**route when push notification is opened**” from Github](https://github.com/react-navigation/react-navigation/issues/742#issuecomment-405592288) and [“**Resolve Javascript Promise outside function scope**” from Stack Overflow](https://stackoverflow.com/questions/26150232/resolve-javascript-promise-outside-function-scope)

**[NavigationService.js](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/NavigationService.js)**
```js
var _setNavigation;  
const _getNavigation = new Promise((resolve, reject) => {  
  _setNavigation = resolve;  
});  
function setTopLevelNavigator(navigatorRef) {  
  _setNavigation(navigatorRef);  
}  
async function navigate(routeName, params) {  
  const navigator = await _getNavigation;  
  navigator.dispatch(  
    NavigationActions.navigate({  
      routeName,  
      params,  
    }),  
  );  
}  
export default { navigate, setTopLevelNavigator };
```

If the **`setTopLevelNavigator`**  function has not run yet, the **`navigate`** function will wait for it to be run.

> Note: I come to Javascript having primarily used other languages with more extensive concurrency features. If you know of a better way to implement [`NavigationService`](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/NavigationService.js), let me know.

====================

The last step is to [set **`android:launchMode`** of the `.MainActivity` to **`singleTop`**](https://developer.android.com/guide/components/activities/tasks-and-back-stack#ManifestForTasks) in [`android\app\src\main\AndroidManifest.xml`](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/android/app/src/main/AndroidManifest.xml#L6)

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.amplifyandroidpush">  
  <application android:name=".MainApplication"  ...>  
    <activity android:name=".MainActivity"  
              ...  
              android:launchMode="singleTop"\>  
...
```

**If you don’t do this, several push notification corner cases will break react-navigation**. For example, navigation will not work if 1) the app is closed, 2) a push notification is received, 3) the app is opened by clicking on the icon, and 4) the push notification is clicked.

The reason is that step 3 (open app with icon) launches the app, but then step 4 (click push notification) navigates to the [`NotificationScreen`](https://github.com/dantasfiles/AmplifyAndroidPush/blob/react-navigation-4/src/NotificationScreen.js), then relaunches the app. By [setting the `android:launchMode` to `singleTop`](https://developer.android.com/guide/components/activities/tasks-and-back-stack#ManifestForTasks), you prevent the app from re-launching when the push notification is pressed if the app is already open.

# Android InAppBrowser for React Native

## Getting started

`$ npm install https://github.com/neami/react-native-inappbrowser.git --save`

### Mostly automatic installation

`$ react-native link react-native-inappbrowser-android`

### Manual installation

#### Android

1.  Open up `android/app/src/main/java/[...]/MainActivity.java`

- Add `import com.neami.inappbrowser.RNInAppBrowserPackage;` to the imports at the top of the file
- Add `new RNInAppBrowserPackage()` to the list returned by the `getPackages()` method

2.  Append the following lines to `android/settings.gradle`:
    ```
    include ':react-native-inappbrowser-android'
    project(':react-native-inappbrowser-android').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-inappbrowser-android/android')
    ```
3.  Insert the following lines inside the dependencies block in `android/app/build.gradle`:
    ```
      implementation project(':react-native-inappbrowser-android')
    ```

## Usage

| Methods       | Action                                                                         |
| ------------- | ------------------------------------------------------------------------------ |
| `open`        | Opens the url with Safari in a modal on Chrome in a new custom tab on Android. |
| `close`       | Dismisses the system's presented web browser                                   |
| `openAuth`    | Opens the url with Safari in a modal on Chrome in a new custom tab on Android. |
| `closeAuth`   | Dismisses the current authentication session                                   |
| `isAvailable` | Detect if the device supports this plugin                                      |

### Demo

```javascript
import InAppBrowser from 'react-native-inappbrowser-android';

...
  async openLink() {
    try {
      await InAppBrowser.isAvailable()
      InAppBrowser.open('https://www.google.com', {
        // Android Properties
        showTitle: true,
        toolbarColor: '#6200EE',
        secondaryToolbarColor: 'black',
        enableUrlBarHiding: true,
        enableDefaultShare: true,
        forceCloseOnRedirection: false,
        // Specify full animation resource identifier(package:anim/name)
        // or only resource name(in case of animation bundled with app).
        animations: {
          startEnter: 'slide_in_right',
          startExit: 'slide_out_left',
          endEnter: 'slide_in_right',
          endExit: 'slide_out_left',
        },
        headers: {
          'my-custom-header': 'my custom header value'
        },
      }).then((result) => {
        Alert.alert(JSON.stringify(result))
      })
    } catch (error) {
      Alert.alert(error.message)
    }
  }
...
```

### Authentication Flow using Deep Linking

- utilities.js

```javascript
import { Platform } from "react-native";
export const getDeepLink = (path = "") => {
  const scheme = "my-scheme";
  const prefix =
    Platform.OS == "android" ? `${scheme}://my-host/` : `${scheme}://`;
  return prefix + path;
};
```

- App.js ([Using react-navigation with Deep Linking](https://reactnavigation.org/docs/en/deep-linking.html))

```javascript
import { Root } from 'native-base'
import { getDeepLink } from './utilities'
import { createStackNavigator } from 'react-navigation'

const Main = createStackNavigator(
  {
    LoginComponent: { screen: LoginComponent },
    HomeComponent: { screen: HomeComponent },
    SplashComponent: { //Redirect users to the Home page if they are authenticated, otherwise to Login page...
      screen: SplashComponent,
      path: 'callback/' //Deep linking to get the auth_token
    }
  },
  {
    index: 0,
    initialRouteName: 'SplashComponent',
    headerMode: 'none'
  }
)
...
  render() {
    return (
      <Root>
        <Main uriPrefix={getDeepLink()} />
      </Root>
    )
  }
...
```

- LoginComponent

```javascript
import { Linking } from 'react-native'
import InAppBrowser from 'react-native-inappbrowser-android'
import { getDeepLink } from './utilities'
...
  async onLogin() {
    const deepLink = getDeepLink("callback")
    const url = `https://my-auth-login-page.com?redirect_uri=${deepLink}`
    try {
      await InAppBrowser.isAvailable()
      InAppBrowser.openAuth(url, deepLink, {
        // Android Properties
        showTitle: false,
        enableUrlBarHiding: true,
        enableDefaultShare: true,
      }).then((response) => {
        if (response.type === 'success' &&
          response.url) {
          Linking.openURL(response.url)
        }
      })
    } catch (error) {
      Linking.openURL(url)
    }
  }
...
```

- SplashComponent

```javascript
...
  componentWillMount() {
    const { navigation } = this.props
    const { state: { params } } = navigation
    const { access_token } = params || {}

    if (access_token) {
      // Opened by deep linking, the user is authenticated
      // Redirect to the Home page
    }
    else {
      // Detect if the stored token is still valid
      // And redirect the user to Home or Login page
    }
  }
...
```

### StatusBar

The StatusBar will keep the last one provided in your app. So if the StatusBar is `dark-content` before you open the browser this will keep it. If you want to change before opening you can do something like

```javascript
  async openInBrowser(url) {
    try {
      StatusBar.setBarStyle('dark-content')
      await InAppBrowser.open(url)
    } catch (error) {
      Alert.alert(error.message);
    }
  })
```

### Credits

- [InAppBrowser for React Native](https://github.com/proyecto26/react-native-inappbrowser)
- [React Native Custom Tabs: Chrome Custom Tabs for React Native](https://github.com/droibit/react-native-custom-tabs)

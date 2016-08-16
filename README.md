# react-native-redux-routing

An exquisitely crafted routing component for Redux based React Native applications.

Providing a consistent user interface for both iOS and Android.

![](https://cloud.githubusercontent.com/assets/8536244/17709163/a7a35d32-641a-11e6-9047-7e2fdd05db72.png)

## Install

`npm install -S react-native-redux-routing`

## Getting Started

Your `Application.js` should looks like below:

```jsx
import React from 'react'

import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'

import {
  actions as routerActions,
  Router,
  Route
} from 'react-native-redux-routing'

import { SplashPage, MainPage } from './pages'

export default connect(
  state => ({
    router: state.router,
  }),
  dispatch => ({
    actions: bindActionCreators(routerActions, dispatch),
  })
)(class extends React.Component {

  render() {
    const config = {
      statusBarStyle: 'light-content',
      renderNavigationView: () => <NavigationDrawer />,
      accentColor: '#C2185B',
    }
    return (
      <Router {...this.props} config={config} initialRoute="splash">
        <Route id="splash" component={SplashPage} immersive={true} />
        <Route id="main" component={MainPage} />
      </Router>
    )
  }

})
```

If you want to use the router along with your own action creators / reducers, add the `mergeProps` function to your route component:

```jsx
import React from 'react'
import { View, Text } from 'react-native'

import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'
import * as userActions from '../actions/userActions'

const mergeProps = (stateProps, dispatchProps, ownProps) => ({
  ...stateProps,
  router: ownProps.router,
  actions: {
    ...dispatchProps.actions,
    ...ownProps.actions,
  }
}

export default connect(
  state => ({
    user: state.user,
  }),
  dispatch => ({
    actions: bindActionCreators(userActions, dispatch)
  }),
  mergeProps)
)(class extends React.Component {

  componentWillMount() {
    const ok = this.props.actions.doSomethingForUser()
    if (ok) {
      const name = this.props.user.name
      this.props.actions._setNavTitle(`Hello ${name}`)
    }
    setTimeout(() => this.props.actions._setNavTitle('WOW'), 2000)
  }

  render() {
    return (
      <View style={{ flex: 1 }}>
        <Text>Current Page: {this.props.router.navTitle}</Text>
      </View>
    )
  }

})
```

## Properties

#### For the `<Router/>` element:

You must set the `initialRoute` property to get the router working.

You can set the `config` property to pass in your custom configurations.

#### For the `<Route/>` element:

You must set the `id` property which is unique to each route.

You must set the `component` property for which class should be rendered.

You can set the `immersive` property to true to hide the app bar (including navigation drawer).

## API

All router-provided actions starts with an underscore in order to prevent possible conflictions.

#### `this.props.actions._navigate(routeId, reset = false)`

```jsx
this.props.actions._navigate('settings') // Push the "settings" route to the routes stack
this.props.actions._navigate(-1) // Pop the last route in the routes stack
this.props.actions._navigate('home', true) // Reset the routes stack and navigate to "home" route
```

#### `this.props.actions._setNavAction(action = { renderer, handler })`

```jsx
this.props.actions._setNavAction({
  renderer: () => <Text>123</Text>, // Function that returns a React element
  handler: () => alert('clicked'), // Function that will triggers when the rendered element was pressed
})
```

#### `this.props.actions._setNavTitle(title)`

```jsx
this.props.actions._setNavTitle('Page Title #' + 1) // Set the page title that shows on the app bar
```

#### `this.props.actions._openDrawer()`

```jsx
this.props.actions._openDrawer() // Open the navigation drawer
```

#### `this.props.actions._closeDrawer()`

```jsx
this.props.actions._closeDrawer() // Close the navigation drawer
```

The dispatchable actions are listed below:

```jsx
import { types as routerTypes } from 'react-native-redux-routing'

dispatch({
  type: routerTypes.PUSH_ROUTE,
  route: 'settings',
})

dispatch({
  type: routerTypes.POP_ROUTE,
})

dispatch({
  type: routerTypes.SET_NAV_ACTION,
  renderer: rendererFunc,
  handler: handlerFunc,
})

dispatch({
  type: routerTypes.SET_NAV_TITLE,
  title: 'New Title',
})

dispatch({
  type: routerTypes.OPEN_DRAWER,
})

dispatch({
  type: routerTypes.CLOSE_DRAWER,
})
```

## Configurations

You can modify the default configuration for the router by passing `config` property into its properties.

The default configuration are listed below:

```jsx
const defaultConfig = {
  renderNavigationView: () => null,
  statusBarStyle: 'default',
  accentColor: '#E0E0E0',
}
```

<table>
  <tr>
    <th>Property</th>
    <th>Type</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>renderNavigationView</td>
    <td>Function</td>
    <td>Function that returns a React element.</td>
  </tr>
  <tr>
    <td>statusBarStyle</td>
    <td>String</td>
    <td>
      Indicates the theme should be dark or light.<br>
      Enum: "default", "light-content"
    </td>
  </tr>
  <tr>
    <td>accentColor</td>
    <td>String</td>
    <td>
      Sets the accent color of the application,<br>
      must be a solid color starting with #.
    </td>
  </tr>
</table>


## Setting up navigation drawer

The drawer layout uses [react-native-drawer-layout](https://github.com/iodine/react-native-drawer-layout) module, you can setup your own navigation drawer view renderer by setting `renderNavigationView` property in the router config object.

```jsx
render() {
  const config = {
    statusBarStyle: 'light-content',
    renderNavigationView: () => <NavigationDrawer />,
    accentColor: '#00695C',
  }
  return (
    <Router {...this.props} config={config} initialRoute="calendar">
      ...
    </Router>
  )
}
```

![](https://cloud.githubusercontent.com/assets/8536244/17712112/26ece3b8-6427-11e6-94e2-bc63cc3eb726.png)

## Adding navigation action dynamically

The example below shows how to adding a camera button dynamically:

```jsx
componentDidMount() {
  this.props.actions._setNavAction({
    renderer: () => (
      <View style={{ alignItems: 'center', flexDirection: 'row' }}>
        <Icon name='camera-alt' style={{ color: '#fff', fontSize: 24 }} />
      </View>
    ),
    handler: () => {
      const date = new Date
      alert('Today is ' + date.toString())
    }
  })
}
```

## Theming

You can theme your application easily by setting the `accentColor` property in the router config object.

The default value of `statusBarStyle` is `"default"` which indicates a light theme, change it to `"light-content"` for dark theme.

![](https://cloud.githubusercontent.com/assets/8536244/17713024/5d4cb15a-642b-11e6-973c-36824572d690.png)

## Credits

Thanks to [react-native-drawer-layout](https://github.com/iodine/react-native-drawer-layout) and [react-native-router-redux](https://github.com/Qwikly/react-native-router-redux) for their awesome work.

[![NPM](https://img.shields.io/npm/v/@3m1/service-worker-updater.svg)](https://www.npmjs.com/package/@3m1/service-worker-updater)
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)
[![Test](https://github.com/emibcn/service-worker-updater/actions/workflows/test.js.yml/badge.svg)](https://github.com/emibcn/service-worker-updater/actions/workflows/test.js.yml)
![Coverage](https://raw.githubusercontent.com/emibcn/service-worker-updater/badges/main/test-coverage.svg)
[![BundlePhobia Minified Size](https://badgen.net/bundlephobia/min/@3m1/service-worker-updater)](https://bundlephobia.com/result?p=@3m1/service-worker-updater)
[![BundlePhobia Minzipped Size](https://badgen.net/bundlephobia/minzip/@3m1/service-worker-updater)](https://bundlephobia.com/result?p=@3m1/service-worker-updater)
[![BundlePhobia Dependency Count](https://badgen.net/bundlephobia/dependency-count/@3m1/service-worker-updater)](https://bundlephobia.com/result?p=@3m1/service-worker-updater)
[![BundlePhobia Tree-shaking support](https://badgen.net/bundlephobia/tree-shaking/@3m1/service-worker-updater)](https://bundlephobia.com/result?p=@3m1/service-worker-updater)

# @3m1/service-worker-updater

> Manage Create React App's Service Worker update

If you have opted-in for the `register` callback of `serviceWorkerRegistration` in the `index.js` of the [PWA version of Create React APP](https://create-react-app.dev/docs/making-a-progressive-web-app/), you probably want to allow your users to update the application once a new service worker has been detected.

## How it works
Usually, browsers check for a new service worker version of a PWA every few days, or whenever the user reloads the page. But reloading the page does not necessarily updates the service worker. As the code managing the service worker is usually outside the React components tree, the message of a _new service worker detected_ needs to be passed through another mechanism than props or contexts. Here, we use an event triggered over `document`, which will previously have been added a listener. The component that adds the listener **is** inside the React's components tree, and receives and saves the `resgistration` object for later use in the `onLoadNewServiceWorkerAccept` callback.

## Install

### NPM

```bash
npm install --save @3m1/service-worker-updater
```

### Yarn

```bash
yarn add @3m1/service-worker-updater
```

## Usage

This library is composed by 2 parts:

### `onServiceWorkerUpdate`
Callback to be added to the `serviceWorkerRegistration.register` call on your `index.js`. **This step is mandatory**, or the message will not arrive to your inner component.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import * as serviceWorkerRegistration from './serviceWorkerRegistration';
import { onServiceWorkerUpdate } from '@3m1/service-worker-updater';

// Render the App
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://cra.link/PWA
serviceWorkerRegistration.register({
  onUpdate: onServiceWorkerUpdate
});

// ...
```

If you are already using the `onUpdate` callback, you need to add this callback in there:

```jsx
serviceWorkerRegistration.register({
  onUpdate: (registration) => {
    // Your code goes here
    // ...
    // Then, call this callback:
    onServiceWorkerUpdate(registration);
  }
});
```

### `withServiceWorkerUpdater`
HOC to wrap a component which will receive 2 extra `props`:
- `newServiceWorkerDetected`: boolean indicating if a new version of the service worker has been detected. If `true`, you should offer the user some way to update the app.
- `onLoadNewServiceWorkerAccept`: a callback which needs to be called once the user accepts to update to the new Service Worker. You choose what actions needs to be taken by the user to update the service worker (a button, a link, a countdown, ...). During its execution, **the page will be reloaded** in order to use the newly activated service worker. **WARNING!** Make sure all unsaved changes are saved before executing it.

```jsx
import React from 'react';
import { withServiceWorkerUpdater } from '@3m1/service-worker-updater';

const Updater = (props) => {
  const {newServiceWorkerDetected, onLoadNewServiceWorkerAccept} = props;
  return newServiceWorkerDetected ? (
    <>
      New version detected.
      <button onClick={ onLoadNewServiceWorkerAccept }>Update!</button>
    </>
  ) : null; // If no update is available, render nothing
}

export default withServiceWorkerUpdater(Updater);
```

The message sent to the service worker is `{type: 'SKIP_WAITING'}`, which is the one the [PWA version of Create React APP](https://create-react-app.dev/docs/making-a-progressive-web-app/) expects in order to launch its `self.skipWaiting()` method. If you have a different service worker configuration, you can change it here using the second optional argument:

```jsx
export default withServiceWorkerUpdater( Updater, {message: {myCustomType: 'SKIP_WAITING'} });
```

Just before reloading the page, `'Controller loaded'` will be logged with `console.log`. If you want to change it, do it so:

```jsx
export default withServiceWorkerUpdater( Updater, {log: () => console.warn("App updated!")});
```

## See also
- [React Service Worker](https://www.npmjs.com/package/@medipass/react-service-worker): A headless React component that wraps around the Navigator Service Worker API to manage your service workers. Inspired by Create React App's service worker registration script.
- [Service Worker Updater - React Hook & HOC](https://www.npmjs.com/package/service-worker-updater): This package provides React hook and HOC to check for service worker updates.
- [@loopmode/cra-workbox-refresh](https://www.npmjs.com/package/@loopmode/cra-workbox-refresh): Helper for `create-react-app` v2 apps that use the workbox service worker. Displays a UI that informs the user about updates and recommends a page refresh.

## License

GPL-3.0-or-later © [github.com/emibcn](https://github.com/github.com/emibcn)

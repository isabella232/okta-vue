[@okta/okta-auth-js]: https://github.com/okta/okta-auth-js
[AuthState]: https://github.com/okta/okta-auth-js#authstatemanager

# Migrating

## From version 2.x to 3.x

### Explicitly accepts `oktaAuth` instance from config

From version 3.0, the `okta-vue` plugin starts to explicitly accept [@okta/okta-auth-js][] instance. You will need to replace the [Okta Auth SDK related configurations](https://github.com/okta/okta-auth-js#configuration-reference) with a pre-initialized [oktaAuth][@okta/okta-auth-js] instance.

If you had code like this:

```javascript
import OktaVue from '@okta/okta-vue'

Vue.use(OktaVue, {
  issuer: 'https://{yourOktaDomain}.com/oauth2/default',
  clientId: '{clientId}',
  redirectUri: window.location.origin + '/login/callback',
  scopes: ['openid', 'profile', 'email']
})
```

it should be rewritten as:

```javascript
import { OktaAuth } from '@okta/okta-auth-js'
import OktaVue from '@okta/okta-vue'

const oktaAuth = new OktaAuth({
  issuer: 'https://{yourOktaDomain}.com/oauth2/default',
  clientId: '{clientId}',
  redirectUri: window.location.origin + '/login/callback',
  scopes: ['openid', 'profile', 'email']
})
Vue.use(OktaVue, { oktaAuth })
```

### Full `@okta/okta-auth-js` API is available

`@okta/okta-vue` version 2.x and earlier provided a wrapper around [@okta/okta-auth-js][] but many methods were hidden. Version 3.x replaces `Auth` service with instance of [@okta/okta-auth-js][] for `$auth`, so the full [api](https://github.com/okta/okta-auth-js#api-reference) and all [options](https://github.com/okta/okta-auth-js#configuration-options) are now supported by this SDK. To provide a better experience, several methods which existed on the wrapper have been removed or replaced.

#### `login` is removed

This method called `onAuthRequired`, if it was set in the config options, or `loginRedirect` if no `onAuthRequired` option was set. If you had code that was calling this method, you may either call your `onAuthRequired` function directly or `signInWithRedirect`.

#### `loginRedirect` is replaced by `signInWithRedirect`

`loginRedirect` took 2 parameters: a `fromUri` and `additionalParams`. The replacement method, [signInWithRedirect](https://github.com/okta/okta-auth-js/blob/master/README.md#signinwithredirectoptions) takes only one argument, called `options` which can include a value for `originalUri` which is equivalent to `fromUri`. It is the URL which will be set after the login flow is complete. Other options which were previously set on `additionalParams` can also be set on `options`.

If you had code like this:

```javascript
$auth.loginRedirect('/profile', { scopes: ['openid', 'profile'] });
```

it should be rewritten as:

```javascript
$auth.signInWithRedirect({ originalUri: '/profile', scopes: ['openid', 'profile'] });
```

#### `logout` is replaced by `signOut`

`logout` accepted either a string or an object as options. [signOut](https://github.com/okta/okta-auth-js/blob/master/README.md#signout) accepts only an options object.

If you had code like this:

```javascript
$auth.logout('/goodbye');
```

it should be rewritten as:

```javascript
$auth.signOut({ postLogoutRedirectUri: window.location.orign + '/goodbye' });
```

Note that the value for `postLogoutRedirectUri` must be an absolute URL. This URL must also be on the "allowed list" in your Okta app's configuration. If no options are passed or no `postLogoutRedirectUri` is set on the options object, it will redirect to `window.location.origin` after sign out is complete.

#### `handleAuthentication` is replaced by `handleLoginRedirect`

`handleLoginRedirect` is called by the `LoginCallback` component as the last step of the login redirect authorization flow. It will obtain and store tokens and then call `restoreOriginalUri` which will return the browser to the `originalUri` which was set before the login redirect flow began.

#### `setFromUri` and `getFromUri` have been replaced with `setOriginalUri` and `getOriginalUri`

[setOriginalUri](https://github.com/okta/okta-auth-js#setoriginaluriuri) is used to save the current/pending URL before beginning a redirect flow. There is a new option, [restoreOriginalUri](https://github.com/okta/okta-auth-js#restoreoriginaluri), which can be used to customize the last step of the login redirect flow.

#### `isAuthenticated` will be true if **both** accessToken **and** idToken are valid

If you have a custom `isAuthenticated` function which implements the default logic, you may remove it.

#### `getAccessToken` and `getIdToken` have been changed to synchronous methods

With maintaining in-memory [AuthState][] since [@okta/okta-auth-js][] version 4.1, token values can be accessed in synchronous manner.

#### `getTokenManager` has been removed

You may access the `TokenManager` with the `tokenManager` property:

```javascript
const tokens = $auth.tokenManager.getTokens();
```

#### `authRedirectGuard` has been removed

Guard logic is handled internally in `@okta/okta-vue@3.x`, previous global guard registration should be removed:

```diff
- router.beforeEach(Vue.prototype.$auth.authRedirectGuard())
```

### "Active" token renew

Previously, tokens would only be renewed when they were read from storge. This typically occurred when a user was navigating to a protected route. Now, tokens will be renewed in the background before they expire. If token renew fails, the [AuthState][] will be updated and `isAuthenticated` will be recalculated. If the user is currently on a protected route, they will need to re-authenticate. Set the `onAuthRequired` option to customize behavior when authentication is required. You can set [tokenManager.autoRenew](https://github.com/okta/okta-auth-js/blob/master/README.md#autorenew) to `false` to disable active token renew logic.

### `Auth.handleCallback` is replaced by `LoginCallback` component

`LoginCallback` component is exported from `@okta/okta-vue` since version 3.0.0. You should replace `Auth.handleCallback()` with `LoginCallback` component.

```diff
- import Auth from `@okta/okta-vue`
+ import { LoginCallback } from `@okta/okta-vue`

...

- { path: '/login/callback', component: Auth.handleCallback() }
+ { path: '/login/callback', component: LoginCallback }
```

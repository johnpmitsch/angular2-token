![Angular2-Token](assets/angular2-token-logo.png)

# Angular2-Token
[![Join the chat at https://gitter.im/lynndylanhurley/devise_token_auth](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/angular2-token/Lobby)
[![npm version](https://badge.fury.io/js/angular2-token.svg)](https://badge.fury.io/js/angular2-token)
[![npm downloads](https://img.shields.io/npm/dt/angular2-token.svg)](https://npmjs.org/angular2-token)
[![Build Status](https://travis-ci.org/neroniaky/angular2-token.svg?branch=master)](https://travis-ci.org/neroniaky/angular2-token)
[![Angular 2 Style Guide](https://mgechev.github.io/angular2-style-guide/images/badge.svg)](https://angular.io/styleguide)

## About
Token based authentication service for Angular2 with multiple user support. Angular2-Token works best with the
[devise token auth](https://github.com/lynndylanhurley/devise_token_auth) gem for Rails.

Angular2-Token is currently in Alpha. Any contribution is much appreciated.

## Live Demo
You can try out Angular2-Token [here](https://angular2-token.herokuapp.com/).

The repository can be found [here](https://github.com/neroniaky/angular2-token-example).

## Content
- [Installation](#installation)
- [Configuration](#configuration)
- [Service Methods](#methods)
    - [`.signIn()`](#signin)
    - [`.signOut()`](#signout)
    - [`.registerAccount()`](#registeraccount)
    - [`.deleteAccount()`](#deleteaccount)
    - [`.validateToken()`](#validatetoken)
    - [`.updatePassword()`](#updatepassword)
    - [`.resetPassword()`](#resetpassword)
    - [`.signInOAuth()`](#signinoauth)
    - [`.processOAuthCallback()`](#processoauthcallback)
- [HTTP Service Wrapper](#http-service-wrapper)
- [Multiple User Types](#multiple-user-types)
- [Route Guards](#route-guards)
- [Advanced Usage](#advanced-usage)
    - [`.sendHttpRequest()`](#sendhttprequest)
    - [`.userSignedIn()`](#usersignedin)
    - [`.currentUserType`](#currentusertype)
    - [`.currentUserData`](#currentuserdata)
    - [`.currentAuthData`](#currentauthdata)
- [Development](#development)
    - [Testing](#testing)
    - [Credits](#credits)
    - [License](#license)

## Installation
1. Install Angular2-Token via NPM with
    ```bash
    npm install angular2-token
    ```

2. Import and add `Angular2TokenService` to your main module. `Angular2TokenService` depends on `HttpModule` and `RouterModule`, so make sure you imported them too.
    ```javascript
    import { HttpModule } from '@angular/http';
    import { RouterModule } from '@angular/router';

    import { Angular2TokenService } from 'angular2-token';

    @NgModule({
        imports: [
            BrowserModule,
            HttpModule,
            RouterModule
        ],
        declarations: [ AppComponent ],
        providers:    [ Angular2TokenService ],
        bootstrap:    [ AppComponent ]
    })
    ```

3. Inject `Angular2TokenService` into your main component and call `.init()`.
    ```javascript
    constructor(private _tokenService: Angular2TokenService) {
        this._tokenService.init();
    }
    ```

4. If you are using CORS in your Rails API make sure that `Access-Control-Expose-Headers` includes `access-token`, `expiry`, `token-type`, `uid`, and `client`.
For the rack-cors gem this can be done by adding the following to its config.
More information can be found [here](https://github.com/lynndylanhurley/devise_token_auth#cors)
    ```ruby
    :expose  => ['access-token', 'expiry', 'token-type', 'uid', 'client']
    ```

## Configuration
Configuration options can be passed as `Angular2TokenOptions` via `.init()`.

### Default Configuration
```javascript
constructor(private _tokenService: Angular2TokenService) {
    this._tokenService.init({
        apiPath:                    null,

        signInPath:                 'auth/sign_in',
        signInRedirect:             null,
        signInStoredUrlStorageKey:  null,

        signOutPath:                'auth/sign_out',
        validateTokenPath:          'auth/validate_token',

        registerAccountPath:        'auth',
        deleteAccountPath:          'auth',
        registerAccountCallback:    window.location.href,

        updatePasswordPath:         'auth',
        resetPasswordPath:          'auth/password',
        resetPasswordCallback:      window.location.href,

        oAuthPaths: {
            github:                 'auth/github'
        },
        oAuthCallbackPath:          'oauth_callback',
        oAuthWindowType:            'newWindow',

        userTypes:                  null,

        globalOptions: {
            headers: {
                'Content-Type':     'application/json',
                'Accept':           'application/json'
            }
        }
    });
}
```

| Options                                 | Description                              |
| --------------------------------------- | ---------------------------------------- |
| `apiPath?: string`                      | Sets base path all operations are based on |
| `signInPath?: string`                   | Sets path for sign in                    |
| `signInRedirect?: string`               | Sets redirect path for failed CanActivate |
| `signInStoredUrlStorageKey?: string`    | Sets locale storage key to store URL before displaying signIn page |
| `signOutPath?: string`                  | Sets path for sign out                   |
| `validateTokenPath?: string`            | Sets path for token validation           |
| `registerAccountPath?: string`          | Sets path for account registration       |
| `deleteAccountPath?: string`            | Sets path for account deletion           |
| `registerAccountCallback?: string`      | Sets the path user are redirected to after email confirmation for registration |
| `updatePasswordPath?: string`           | Sets path for password update            |
| `resetPasswordPath?: string`            | Sets path for password reset             |
| `resetPasswordCallback?: string`        | Sets the path user are redirected to after email confirmation for password reset |
| `userTypes?: UserTypes[]`               | Allows the configuration of multiple user types (see [Multiple User Types](#multiple-user-types)) |
| `globalOptions?: GlobalOptions`         | Allows the configuration of global options (see below) |
| `oAuthPaths?: { [key:string]: string }` | Sets paths for sign in with OAuth        |
| `oAuthCallbackPath?:  string`           | Sets path for OAuth sameWindow callback  |
| `oAuthWindowType?:`string`              | Window type for Oauth authentication     |
### Global Options
| Options                               | Description                                     |
| ------------------------------------- | ----------------------------------------------- |
| `headers?: { [key:string]: string; }` | Define custom global headers as hashmap. Be careful when overwriting the default options, `devise token auth` will refuse requests without the `Content-Type`-Header set |

Further information on paths/routes can be found at
[devise token auth](https://github.com/lynndylanhurley/devise_token_auth#usage-tldr)

## Service Methods
Once initialized `Angular2TokenService` offers methods for session management.

### .signIn()
The signIn method is used to sign in the user with email address and password.
The optional parameter `type` specifies the name of UserType used for this session.

`signIn(email: string, password: string, userType?: string): Observable<Response>`

#### Example:
```javascript
this._tokenService.signIn(
    'example@example.org',
    'secretPassword'
).subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .signOut()
The signOut method destroys session and session storage.

`signOut(): Observable<Response>`

#### Example:
```javascript
this._tokenService.signOut().subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .registerAccount()
Sends a new user registration request to the Server.

`registerAccount(email: string, password: string, passwordConfirmation: string, userType?: string): Observable<Response>`

#### Example:
```javascript
this._tokenService.registerAccount(
    'example@example.org',
    'secretPassword',
    'secretPassword'
).subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .deleteAccount()
Deletes the account for the signed in user.

`deleteAccount(): Observable<Response>`

#### Example:
```javascript
this._tokenService.deleteAccount().subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .validateToken()
Validates the current token with the server.

`validateToken(): Observable<Response>`

#### Example:
```javascript
this._tokenService.validateToken().subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .updatePassword()
Updates the password for the logged in user.

`updatePassword(password: string, passwordConfirmation: string, currentPassword?: string, userType?: string): Observable<Response>`

#### Example:
```javascript
this._tokenService.updatePassword(
    'newPassword',
    'newPassword',
    'oldPassword'
).subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .resetPassword()
Request a password reset from the server.

`resetPassword(email: string, userType?: string): Observable<Response>`

#### Example:
```javascript
this._tokenService.updatePassword(
    'example@example.org',
).subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .signInOAuth()
Initiates OAuth authentication flow by redirecting to OAuth path. Currently, it supports two window modes(oAuthWindowType):
'newWindow' (default) and 'sameWindow'.
When `oAuthWindowType` is set to `newWindow`, `.signInOAuth()` opens a new window and returns an observable.

When `oAuthWindowType` is set to 'sameWindow', `.signInOAuth()` returns nothing and redirects user to auth provider.
After successful authentication, it redirects back to `oAuthCallbackPath`. Application router needs to intercept
this route and call `processOAuthCallback()` to fetch `AuthData` from params.

`signInOAuth(oAuthType: string)`

#### Example:

```javascript
this._tokenService.signInOAuth(
'github'
).subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

### .processOAuthCallback()
Fetches AuthData from params sent via OAuth redirection when `oAuthWindowType` is set to `sameWindow`.

`processOAuthCallback()`

#### Example

Callback route:
```javascript
RouterModule.forRoot([
  { path: 'oauth_callback', component: OauthCallbackComponent }
])
```

Callback component:
```javascript
@Component({
  template: ''
})
export class OauthCallbackComponent implements OnInit {
  constructor(private _tokenService: Angular2TokenService) {}

  ngOnInit() {
    this._tokenService.processOAuthCallback();
  }
}
```

## HTTP Service Wrapper
`Angular2TokenService` wraps all standard Angular2 Http Service calls for authentication and token processing. 
If `apiPath` is configured it gets added in front of path.
- `get(path: string): Observable<Response>`
- `post(path: string, data: any): Observable<Response>`
- `put(path: string, data: any): Observable<Response>`
- `delete(path: string): Observable<Response>`
- `patch(path: string, data: any): Observable<Response>`
- `head(path: string): Observable<Response>`
- `options(path: string): Observable<Response>`

#### Example:
```javascript
this._tokenService.get('my-resource/1').map(res => res.json()).subscribe(
    res =>      console.log(res),
    error =>    console.log(error)
);
```

## Multiple User Types
An array of `UserType` can be passed in `Angular2TokenOptions` during `init()`.
The user type is selected during sign in and persists until sign out.
`.currentUserType` returns the currently logged in user.

#### Example:
```javascript
this._tokenService.init({
    userTypes: [
        { name: 'ADMIN', path: 'admin' },
        { name: 'USER', path: 'user' }
    ]
});

this._tokenService.signIn(
    'example@example.com',
    'secretPassword',
    'ADMIN'
)

this._tokenService.currentUserType; // ADMIN
```

## Route Guards
Angular2-Token implements the `CanActivate` interface, so it can directly be used as a route guard. 
If the `signInRedirect` option is set the user will be redirected on a failed (=false) CanActivate using `Router.navigate()`.
It currently does not distinguish between user types.

#### Example:
```javascript
const routerConfig: Routes = [
    { 
        path: '', 
        component: PublicComponent 
    }, {
        path: 'restricted', 
        component: RestrictedComponent, 
        canActivate: [Angular2TokenService] 
    }
];
```

## Advanced Usage
More advanced methods can be used if a higher degree of customization is required.

### .sendHttpRequest()
More customized requests can be send with the `.sendHttpRequest()`-function. It accepts the RequestOptions-Class. 
More information can be found in the Angular2 API Reference [here](https://angular.io/docs/ts/latest/api/http/index/RequestOptions-class.html).

`sendHttpRequest(options: RequestOptions): Observable<Response>`

#### Example:
```javascript
this.sendHttpRequest(new RequestOptions({
    method: RequestMethod.Post,
    url:    'my-resource/1',
    data:   mydata
}));
```

### .userSignedIn()
Returns `true` if a user is signed in. It does not distinguish between user types.

`userSignedIn(): boolean`

### .currentUserType
Returns current user type as string like specified in the options.

`get currentUserType(): string`

### .currentUserData
Returns current user data as returned by devise token auth. 
This variable is `null` after page reload until the `.validateToken()` call is answerd by the backend.

`get currentUserData(): UserData`

### .currentAuthData
Returns current authentication data which are used to set auth headers.

`get currentAuthData(): AuthData`

### Redirect original requested URL
If you want to redirect to the protected URL after signing in, you need to set `signInStoredUrlStorageKey` and in your code you can do something like this

```js
this._tokenService.signIn(
    'example@example.org',
    'secretPassword'
).subscribe(
    res => {
        // You have to add Router DI in your component
        this.router.navigateByUrl(localStorage.getItem('redirectTo'));
    },
    error =>    console.log(error)
);
```

## Development
If the package is installed from Github specified in the package.json, you need to build the package locally.

```bash
cd ./node_modules/angular2-token
npm install
npm run build
```

### Testing
```bash
npm test
```

### Credits
Test config files based on [Angular2 Webpack Starter](https://github.com/AngularClass/angular2-webpack-starter) by AngularClass

### License
The MIT License (see the [LICENSE](https://github.com/neroniaky/angular2-token/blob/master/LICENSE) file for the full text)

everyauth
==========

Authentication and authorization (password, facebook, & more) for your node.js Connect and Express apps.

There is a NodeTuts screencast of everyauth [here](http://nodetuts.com/tutorials/26-starting-with-everyauth.html#video)

So far, `everyauth` enables you to login via:

- `password`
- `OpenId` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Credits [RocketLabs Development](https://github.com/rocketlabsdev), [Andrew Mee](https://github.com/starfishmod), [Brian Noguchi](https://github.com/bnoguchi))
  - `Google Hybrid` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Credits [RocketLabs Development](https://github.com/rocketlabsdev))
- OAuth
  - `twitter`
  - `linkedin`
  - `yahoo`
  - `readability` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Credits [Alfred Nerstu](https://github.com/alfrednerstu))
  - `dropbox` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Credits [Torgeir](https://github.com/torgeir))
  - `justin.tv` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Credits [slickplaid](https://github.com/slickplaid))
  - `vimeo` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Credits [slickplaid](https://github.com/slickplaid))
  - `tumblr`
- OAuth2
  - `facebook`
  - `github`
  - `instagram`
  - `foursquare`
  - `google`
  - `gowalla` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Credits [Andrew Kramolisch](https://github.com/andykram))
  - `37signals` (Basecamp, Highrise, Backpack, Campfire)
- `box` (Box.net)
- `LDAP` (experimental; not production-tested)

`everyauth` is:

- **Modular** - We have you covered with Facebook and Twitter 
  OAuth logins, basic login/password support, and modules 
  coming soon for beta invitation support and more.
- **Easily Configurable** - everyauth was built with powerful
  configuration needs in mind. Configure an authorization strategy 
  in a straightforward, easy-to-read & easy-to-write approach, 
  with as much granularity as you want over the steps and 
  logic of your authorization strategy.
- **Idiomatic** - The syntax for configuring and extending your authorization strategies are
  idiomatic and chainable.


## Installation
    $ npm install everyauth


## Quick Start
Using everyauth comes down to just 2 simple steps if using Connect
or 3 simple steps if using Express:

1. **Choose and Configure Auth Strategies** - Find the authentication strategy
   you desire in one of the sections below. Follow the configuration
   instructions.
2. **Add the Middleware to Connect**
        
    ```javascript
    var everyauth = require('everyauth');
    // Step 1 code goes here

    // Step 2 code
    var connect = require('connect');
    var app = connect(
        connect.bodyParser()
      , connect.cookieParser()
      , connect.session({secret: 'mr ripley'})
      , everyauth.middleware()
      , connect.router(routes)
    );
    ```
3. **Add View Helpers to Express**
    
    ```javascript        
    // Step 1 code
    // ...
    // Step 2 code
    // ...

    // Step 3 code
    everyauth.helpExpress(app);

    app.listen(3000);
    ```
    
    For more about what view helpers `everyauth` adds to your app, see the section
    titled "Express Helpers" near the bottom of this README.

## Example Application

There is an example application at [./example](https://github.com/bnoguchi/everyauth/tree/master/example)

To run it:

    $ cd example
    $ node server.js

**Important** - Some OAuth Providers do not allow callbacks to localhost, so you will need to create a `localhost`
alias called `local.host`. Make sure you set up your /etc/hosts so that 127.0.0.1 is also 
associated with 'local.host'. So inside your /etc/hosts file, one of the lines will look like:

    127.0.0.1	localhost local.host

Then point your browser to [http://local.host:3000](http://local.host:3000)

## Tests

First, spin up the example server (See last section "Example Application").

Then,

    $ make test

## Logging Out

If you integrate `everyauth` with `connect`, then `everyauth` automatically
sets up a `logoutPath` at `GET` `/logout` for your app. It also
sets a default handler for your logout route that clears your session
of auth information and redirects them to '/'.

To over-write the logout path:

```javascript
everyauth.everymodule.logoutPath('/bye');
```

To over-write the logout redirect path:

```javascript
everyauth.everymodule.logoutRedirectPath('/navigate/to/after/logout');
```

To over-write the logout handler:

```javascript
everyauth.everymodule.handleLogout( function (req, res) {
  // Put you extra logic here
  
  req.logout(); // The logout method is added for you by everyauth, too
  
  // And/or put your extra logic here
  
  res.writeHead(303, { 'Location': this.logoutRedirectPath() });
  res.end();
});
```

## Setting up Facebook Connect

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.facebook
  .appId('YOUR APP ID HERE')
  .appSecret('YOUR APP SECRET HERE')
  .handleAuthCallbackError( function (req, res) {
    // If a user denies your app, Facebook will redirect the user to
    // /auth/facebook/callback?error_reason=user_denied&error=access_denied&error_description=The+user+denied+your+request.
    // This configurable route handler defines how you want to respond to
    // that.
    // If you do not configure this, everyauth renders a default fallback
    // view notifying the user that their authentication failed and why.
  })
  .findOrCreateUser( function (session, accessToken, accessTokExtra, fbUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.facebook
  .entryPath('/auth/facebook')
  .callbackPath('/auth/facebook/callback')
  .scope('email')                // Defaults to undefined
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.facebook.scope(); // undefined
everyauth.facebook.entryPath(); // '/auth/facebook'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.facebook.configurable();
```

#### Dynamic Facebook Connect Scope

Facebook provides many different 
[permissions](http://developers.facebook.com/docs/authentication/permissions/)
for which your app can ask your user. This is bundled up in the `scope` query
paremter sent with the oauth request to Facebook. While your app may require 
several different permissions from Facebook, Facebook recommends that you only
ask for these permissions incrementally, as you need them. For example, you might
want to only ask for the "email" scope upon registration. At the same time, for
another user, you may want to ask for "user_status" permissions because they
have progressed further along in your application.

`everyauth` enables you to specify the "scope" dynamically with a second
variation of the configurable `scope`. In addition to the first variation
that looks like:

```javascript
everyauth.facebook
  .scope('email,user_status');
```

you can have greater dynamic control over "scope" via the second variation of `scope`:

```javascript
everyauth.facebook
  .scope( function (req, res) {
    var session = req.session;
    switch (session.userPhase) {
      case 'registration':
        return 'email';
      case 'share-media':
        return 'email,user_status';
    }
  });

```

## Setting up Twitter OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.twitter
  .consumerKey('YOUR CONSUMER ID HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (session, accessToken, accessTokenSecret, twitterUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

**Important** - Some developers forget to do the following, and it causes them to have issues with `everyauth`.
Please make sure to do the following: When you set up your app at http://dev.twitter.com/, make sure that your callback url is set up to
include that path '/auth/twitter/callback/'. In general, when dealing with OAuth or OAuth2 modules
provided by `everyauth`, the default callback path is always set up to follow the pattern
'/auth/#{moduleName}/callback', so just ensure that you configure your OAuth settings accordingly with
the OAuth provider -- in this case, the "Edit Application Settings" section for your app at http://dev.twitter.com.

Alternatively, you can specify the callback url at the application level by configuring `callbackPath` (which
has a default configuration of "/auth/twitter/callback"):

```javascript
everyauth.twitter
  .consumerKey('YOUR CONSUMER ID HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .callbackPath('/custom/twitter/callback/path')
  .findOrCreateUser( function (session, accessToken, accessTokenSecret, twitterUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');
```

So if your hostname is `example.com`, then this configuration will over-ride the `dev.twitter.com` callback url configuration.
Instead, Twitter will redirect back to `example.com/custom/twitter/callback/path` in the example just given above.

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.twitter
  .entryPath('/auth/twitter')
  .callbackPath('/auth/twitter/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.twitter.callbackPath(); // '/auth/twitter/callback'
everyauth.twitter.entryPath(); // '/auth/twitter'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.twitter.configurable();
```

## Setting up Password Authentication

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.password
  .getLoginPath('/login') // Uri path to the login page
  .postLoginPath('/login') // Uri path that your login form POSTs to
  .loginView('a string of html; OR the name of the jade/etc-view-engine view')
  .authenticate( function (login, password) {
    // Either, we return a user or an array of errors if doing sync auth.
    // Or, we return a Promise that can fulfill to promise.fulfill(user) or promise.fulfill(errors)
    // `errors` is an array of error message strings
    //
    // e.g., 
    // Example 1 - Sync Example
    // if (usersByLogin[login] && usersByLogin[login].password === password) {
    //   return usersByLogin[login];
    // } else {
    //   return ['Login failed'];
    // }
    //
    // Example 2 - Async Example
    // var promise = this.Promise()
    // YourUserModel.find({ login: login}, function (err, user) {
    //   if (err) return promise.fulfill([err]);
    //   promise.fulfill(user);
    // }
    // return promise;
  })
  .loginSuccessRedirect('/') // Where to redirect to after a login
  
    // If login fails, we render the errors via the login view template,
    // so just make sure your loginView() template incorporates an `errors` local.
    // See './example/views/login.jade'

  .getRegisterPath('/register') // Uri path to the registration page
  .postRegisterPath('/register') // The Uri path that your registration form POSTs to
  .registerView('a string of html; OR the name of the jade/etc-view-engine view')
  .validateRegistration( function (newUserAttributes) {
    // Validate the registration input
    // Return undefined, null, or [] if validation succeeds
    // Return an array of error messages (or Promise promising this array)
    // if validation fails
    //
    // e.g., assuming you define validate with the following signature
    // var errors = validate(login, password, extraParams);
    // return errors;
    //
    // The `errors` you return show up as an `errors` local in your jade template
  })
  .registerUser( function (newUserAttributes) {
    // This step is only executed if we pass the validateRegistration step without
    // any errors.
    //
    // Returns a user (or a Promise that promises a user) after adding it to
    // some user store.
    //
    // As an edge case, sometimes your database may make you aware of violation
    // of the unique login index, so if this error is sent back in an async
    // callback, then you can just return that error as a single element array
    // containing just that error message, and everyauth will automatically handle
    // that as a failed registration. Again, you will have access to this error via
    // the `errors` local in your register view jade template.
    // e.g.,
    // var promise = this.Promise();
    // User.create(newUserAttributes, function (err, user) {
    //   if (err) return promise.fulfill([err]);
    //   promise.fulfill(user);
    // });
    // return promise;
    //
    // Note: Index and db-driven validations are the only validations that occur 
    // here; all other validations occur in the `validateRegistration` step documented above.
  })
  .registerSuccessRedirect('/'); // Where to redirect to after a successful registration

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.password
  .loginFormFieldName('login')       // Defaults to 'login'
  .passwordFormFieldName('password') // Defaults to 'password'
  .loginLayout('custom_login_layout') // Only with `express`
  .registerLayout('custom reg_layout') // Only with `express`
  .loginLocals(fn);                    // See Recipe 3 below
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.password.loginFormFieldName();    // 'login'
everyauth.password.passwordFormFieldName(); // 'password'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.password.configurable();
```

### Password Recipe 1: Extra registration data besides login + password

Sometimes your registration will ask for more information from the user besides the login and password.

For this particular scenario, you can configure the optional step, `extractExtraRegistrationParams`.

```javascript
everyauth.password.extractExtraRegistrationParams( function (req) {
  return {
      phone: req.body.phone
    , name: {
          first: req.body.first_name
        , last: req.body.last_name
      }
  };
});
```

### Password Recipe 2: Logging in with email or phone number

By default, `everyauth` uses the field and user key name `login` during the
registration and login process.

Sometimes, you want to use `email` or `phone` instead of `login`. Moreover,
you also want to validate `email` and `phone` fields upon registration.

`everyauth` provides an easy way to do this:

```javascript
everyauth.password.loginWith('email');

// OR

everyauth.password.loginWith('phone');
```

With simple login configuration like this, you get email (or phone) validation
in addition to renaming of the form field and user key corresponding to what
otherwise would typically be referred to as 'login'.

### Password Recipe 3: Adding additional view local variables to login and registration views

If you are using `express`, you are able to pass variables from your app
context to your view context via local variables. `everyauth` provides
several convenience local vars for your views, but sometimes you will want
to augment this set of local vars with additional locals.

So `everyauth` also provides a mechanism for you to do so via the following
configurables:

```javascript
everyauth.password.loginLocals(...);
everyauth.password.registerLocals(...);
```

`loginLocals` and `registerLocals` configuration have symmetrical APIs, so I
will only cover `loginLocals` here to illustrate how to use both.

You can configure this parameter in one of *3* ways. Why 3? Because there are 3 types of ways that you can retrieve your locals.

1. Static local vars that never change values:
   
       ```javascript
       everyauth.password.loginLocals({
         title: 'Login'
       });
       ```
2. Dynamic synchronous local vars that depend on the incoming request, but whose values are retrieved synchronously
   
       ```javascript
       everyauth.password.loginLocals( function (req, res) {
         var sess = req.session;
         return {
           isReturning: sess.isReturning
         };
       });
       ```
3. Dynamic asynchronous local vars
   
       ```javascript
       everyauth.password.loginLocals( function (req, res, done) {
         asyncCall( function ( err, data) {
           if (err) return done(err);
           done(null, {
             title: il8n.titleInLanguage('Login Page', il8n.language(data.geo))
           });
         });
       });
       ```

### Password Recipe 4: Customize Your Registration Validation

By default, `everyauth.password` automatically

- validates that the login (or email or phone, depending on what you authenticate with -- see Password Recipe 2) is present in the login http request, 
- validates that the password is present
- validates that an email login is a correctly formatted email
- validates that a phone login is a valid phone number

If any of these validations fail, then the appropriate errors are generated and accessible to you in your view via the `errors` view local variable.

If you want to add additional validations beyond this, you can do so by configuring the step, `validateRegistration`:

```javascript
everyauth.password
  .validateRegistration( function (newUserAttributes, baseErrors) {
    // Here, newUserAttributes is the hash of parameters extracted from the incoming request.
    // baseErrors is the array of errors generated by the default automatic validation outlined above
    //   in this same recipe.

    // First, validate your errors. Here, validateUser is a made up function
    var moreErrors = validateUser( newUserAttributes );
    if (moreErrors.length) baseErrors.push.apply(baseErrors, moreErrors);

    // Return the array of errors, so your view has access to them.
    return baseErrors;
  });
```

## Setting up GitHub OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.github
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .findOrCreateUser( function (session, accessToken, , accessTokenExtra, githubUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:
  
```javascript  
everyauth.github
  .entryPath('/auth/github')
  .callbackPath('/auth/github/callback')
  .scope('repo'); // Defaults to undefined
                  // Can be set to a combination of: 'user', 'public_repo', 'repo', 'gist'
                  // For more details, see http://develop.github.com/p/oauth.html
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.github.scope(); // undefined
everyauth.github.entryPath(); // '/auth/github'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.github.configurable();
```

## Setting up Instagram OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.instagram
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .findOrCreateUser( function (session, accessToken, accessTokenExtra, instagramUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.instagram
  .entryPath('/auth/instagram')
  .callbackPath('/auth/instagram/callback')
  .scope('basic') // Defaults to 'basic'
                  // Can be set to a combination of: 'basic', 'comments', 'relationships', 'likes'
                  // For more details, see http://instagram.com/developer/auth/#scope
  .display(undefined); // Defaults to undefined; Set to 'touch' to see a mobile optimized version
                       // of the instagram auth page
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.instagram.callbackPath(); // '/auth/instagram/callback'
everyauth.instagram.entryPath(); // '/auth/instagram'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.instagram.configurable();
```

## Setting up Foursquare OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.foursquare
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .findOrCreateUser( function (session, accessToken, accessTokenExtra, foursquareUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.foursquare
  .entryPath('/auth/foursquare')
  .callbackPath('/auth/foursquare/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.foursquare.callbackPath(); // '/auth/foursquare/callback'
everyauth.foursquare.entryPath(); // '/auth/foursquare'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.foursquare.configurable();
```

## Setting up LinkedIn OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.linkedin
  .consumerKey('YOUR CONSUMER ID HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (session, accessToken, accessTokenSecret, linkedinUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.linkedin
  .entryPath('/auth/linkedin')
  .callbackPath('/auth/linkedin/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.linkedin.callbackPath(); // '/auth/linkedin/callback'
everyauth.linkedin.entryPath(); // '/auth/linkedin'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.linkedin.configurable();
```

## Setting up Google OAuth2

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.google
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .scope('https://www.google.com/m8/feeds') // What you want access to
  .handleAuthCallbackError( function (req, res) {
    // If a user denies your app, Google will redirect the user to
    // /auth/facebook/callback?error=access_denied
    // This configurable route handler defines how you want to respond to
    // that.
    // If you do not configure this, everyauth renders a default fallback
    // view notifying the user that their authentication failed and why.
  })
  .findOrCreateUser( function (session, accessToken, accessTokenExtra, googleUserMetadata) {
    // find or create user logic goes here
    // Return a user or Promise that promises a user
    // Promises are created via
    //     var promise = this.Promise();
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.google
  .entryPath('/auth/google')
  .callbackPath('/auth/google/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.google.scope(); // undefined
everyauth.google.entryPath(); // '/auth/google'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.google.configurable();
```

## Setting up Gowalla OAuth2

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.gowalla
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .handleAuthCallbackError( function (req, res) {
    // TODO - Update this documentation
    // This configurable route handler defines how you want to respond to
    // a response from Gowalla that something went wrong during the oauth2 process.
    // If you do not configure this, everyauth renders a default fallback
    // view notifying the user that their authentication failed and why.
  })
  .findOrCreateUser( function (session, accessToken, accessTokenExtra, gowallaUserMetadata) {
    // find or create user logic goes here
    // Return a user or Promise that promises a user
    // Promises are created via
    //     var promise = this.Promise();
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.gowalla
  .entryPath('/auth/gowalla')
  .callbackPath('/auth/gowalla/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.gowalla.scope(); // undefined
everyauth.gowalla.entryPath(); // '/auth/gowalla'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.gowalla.configurable();
```

## Setting up 37signals (Basecamp, Highrise, Backpack, Campfire) OAuth2

First, register an app at [integrate.37signals.com](https://integrate.37signals.com).

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth['37signals']
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .handleAuthCallbackError( function (req, res) {
    // TODO - Update this documentation
    // This configurable route handler defines how you want to respond to
    // a response from 37signals that something went wrong during the oauth2 process.
    // If you do not configure this, everyauth renders a default fallback
    // view notifying the user that their authentication failed and why.
  })
  .findOrCreateUser( function (session, accessToken, accessTokenExtra, _37signalsUserMetadata) {
    // find or create user logic goes here
    // Return a user or Promise that promises a user
    // Promises are created via
    //     var promise = this.Promise();
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth['37signals']
  .entryPath('/auth/37signals')
  .callbackPath('/auth/37signals/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth['37signals'].entryPath(); // '/auth/37signals'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth['37signals'].configurable();
```

## Setting up Yahoo OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.yahoo
  .consumerKey('YOUR CONSUMER KEY HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (session, accessToken, accessTokenSecret, yahooUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.yahoo
  .entryPath('/auth/yahoo')
  .callbackPath('/auth/yahoo/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.yahoo.callbackPath(); // '/auth/yahoo/callback'
everyauth.yahoo.entryPath(); // '/auth/yahoo'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.yahoo.configurable();
```

## Setting up Readability OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.readability
  .consumerKey('YOUR CONSUMER KEY HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (sess, accessToken, accessSecret, reader) {
    // find or create user logic goes here
    //
    // e.g.,
    // return usersByReadabilityId[reader.username] || (usersByReadabilityId[reader.username] = reader);
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.readability
  .entryPath('/auth/readability')
  .callbackPath('/auth/readability/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.readability.callbackPath(); // '/auth/readability/callback'
everyauth.readability.entryPath(); // '/auth/readability'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.readability.configurable();
```

## Setting up Dropbox OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.dropbox
  .consumerKey('YOUR CONSUMER KEY HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (sess, accessToken, accessSecret, user) {
    // find or create user logic goes here
    //
    // e.g.,
    // return usersByDropboxId[user.uid] || (usersByDropboxId[user.uid] = user);
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.dropbox
  .entryPath('/auth/dropbox')
  .callbackPath('/auth/dropbox/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.dropbox.callbackPath(); // '/auth/dropbox/callback'
everyauth.dropbox.entryPath(); // '/auth/dropbox'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.dropbox.configurable();
```

## Setting up Justin.tv OAuth

[Sign up for a Justin.tv account](http://www.justin.tv/user/signup) and activate it as a [developer account](http://www.justin.tv/developer/activate) to get your consumer key and secret.

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');
  
everyauth.justintv
  .consumerKey('YOUR CONSUMER KEY HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (sess, accessToken, accessSecret, justintvUser) {
    // find or create user logic goes here
    //
    // e.g.,
    // return usersByJustintvId[justintvUser.id] || (usersByJustintvId[justintvUser.id] = justintvUser);
  })
  .redirectPath('/');
  
var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

The `justintvUser` parameter in the `.findOrCreateUser()` function above returns the `account/whoami` API call

[Justin.tv API Wiki - Account/whoami](http://apiwiki.justin.tv/mediawiki/index.php/Account/whoami)

```javascript
{
   "image_url_huge": "http:\/\/static-cdn.justin.tv\/jtv_user_pictures\/justin-320x240-4.jpg",
   "profile_header_border_color": null,
   "favorite_quotes": "I love Justin.tv",
   "sex": "Male",
   "image_url_large": "http:\/\/static-cdn.justin.tv\/jtv_user_pictures\/justin-125x94-4.jpg",
   "profile_about": "Check out my website:\n\nwww.justin.tv\n",
   "profile_background_color": null,
   "image_url_medium": "http:\/\/static-cdn.justin.tv\/jtv_user_pictures\/justin-75x56-4.jpg",
   "id": 1698,
   "broadcaster": true,
   "profile_url": "http:\/\/www.justin.tv\/justin\/profile",
   "profile_link_color": null,
   "image_url_small": "http:\/\/static-cdn.justin.tv\/jtv_user_pictures\/justin-50x37-4.jpg",
   "profile_header_text_color": null,
   "name": "The JUST UN",
   "image_url_tiny": "http:\/\/static-cdn.justin.tv\/jtv_user_pictures\/justin-33x25-4.jpg",
   "login": "justin",
   "profile_header_bg_color": null,
   "location": "San Francisco"
}
```

You can also configure more parameters (most are set to defaults) via the same chainable API:

```javascript
everyauth.justintv
  .entryPath('/auth/justintv')
  .callbackPath('/auth/justintv/callback');
```

If you want to see what the current value of a configured parameter is, you can do so via:

```javascript
everyauth.justintv.callbackPath(); // '/auth/justintv/callback'
everyauth.justintv.entryPath(); // '/auth/justintv'
```

To see all parameters that are configurable, the following will return an object whose parameter name keys map to description values:

```javascript
everyauth.justintv.configurable();
```

## Setting up Vimeo OAuth

You will first need to sign up for a [developer application](http://vimeo.com/api/applications) to get the consumer key and secret.

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.vimeo
  .consumerKey('YOUR CONSUMER KEY HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (sess, accessToken, accessSecret, user) {
    // find or create user logic goes here
    //
    // e.g.,
    // return usersByVimeoId[user.id] || (usersByVimeoId[user.id] = user);
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.vimeo
  .entryPath('/auth/vimeo')
  .callbackPath('/auth/vimeo/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.vimeo.callbackPath(); // '/auth/vimeo/callback'
everyauth.vimeo.entryPath(); // '/auth/vimeo'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.vimeo.configurable();
```

## Setting up Tumblr OAuth (1.a)

You will first need to [register an app](http://www.tumblr.com/oauth/register) to get the consumer key and secret.
During registration of your new app, enter a "Default callback URL" of "http://<hostname>:<port>/auth/tumblr/callback".
Once you register your app, copy down your "OAuth Consumer Key" and "Secret Key" and proceed below.

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.tumblr
  .consumerKey('YOUR CONSUMER KEY HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (sess, accessToken, accessSecret, user) {
    // find or create user logic goes here
    //
    // e.g.,
    // return usersByTumblrName[user.name] || (usersByTumblrName[user.name] = user);
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.tumblr
  .entryPath('/auth/tumblr')
  .callbackPath('/auth/tumblr/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.tumblr.callbackPath(); // '/auth/tumblr/callback'
everyauth.tumblr.entryPath(); // '/auth/tumblr'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.tumblr.configurable();
```

## Setting up OpenID protocol

OpenID protocol allows you to use an openid auth request. You can read more information about it here http://openid.net/

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.openid
  .myHostname('http://localhost:3000')
  .simpleRegistration({
      "nickname" : true
    , "email"    : true
    , "fullname" : true
    , "dob"      : true
    , "gender"   : true
    , "postcode" : true
    , "country"  : true
    , "language" : true
    , "timezone" : true
  })
	.attributeExchange({
      "http://axschema.org/contact/email"       : "required"
    , "http://axschema.org/namePerson/friendly" : "required"
    , "http://axschema.org/namePerson"          : "required"
    , "http://axschema.org/namePerson/first"    : "required"
    , "http://axschema.org/contact/country/home": "required"
    , "http://axschema.org/media/image/default" : "required"
    , "http://axschema.org/x/media/signature"   : "required"
  })
  .openidURLField('openid_identifier'); //The POST variable used to get the OpenID
  .findOrCreateUser( function(session, openIdUserAttributes) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

## Setting up Google OpenID+OAuth Hybrid protocol

OpenID+OAuth Hybrid protocol allows you to combine an openid auth request with a oauth access request. You can read more information about it here http://code.google.com/apis/accounts/docs/OpenID.html

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.googlehybrid
  .consumerKey('YOUR CONSUMER ID HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .scope(['GOOGLE API SCOPE','GOOGLE API SCOPE'])
  .findOrCreateUser( function(session, userAttributes) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

## Setting up Box.net Auth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.box
  .apiKey('YOUR API KEY')
  .findOrCreateUser( function (sess, authToken, boxUser) {
    // find or create user logic goes here
    //
    // e.g.,
    // return usersByBoxId[user.user_id] || (usersByBoxId[user.user_id] = user);
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.box
  .entryPath('/auth/box')
  .callbackPath('/auth/box/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.box.callbackPath(); // '/auth/box/callback'
everyauth.box.entryPath(); // '/auth/box'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.box.configurable();
```

## Setting up LDAP

The LDAP module is still in development. Do not use it in production yet.

Install OpenLDAP client libraries:

    $ apt-get install slapd ldap-utils

Install [node-ldapauth](https://github.com/joewalnes/node-ldapauth):

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.ldap
  .host('your.ldap.host')
  .port(389)

  // The `ldap` module inherits from the `password` module, so 
  // refer to the `password` module instructions several sections above
  // in this README.
  // You do not need to configure the `authenticate` step as instructed
  // by `password` because the `ldap` module already does that for you.
  // Moreover, all the registration related steps and configurable parameters
  // are no longer valid
  .getLoginPath(...)
  .postLoginPath(...)
  .loginView(...)
  .loginSuccessRedirect(...);

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

## Accessing the User

If you are using `express` or `connect`, then `everyauth` 
provides an easy way to access the user as:

- `req.user` from your app server
- `everyauth.user` via the `everyauth` helper accessible from your `express` views.
- `user` as a helper accessible from your `express` views

To access the user, configure `everyauth.everymodule.findUserById`.
For example, using [mongoose](http://github.com/LearnBoost/mongoose):

```javascript
everyauth.everymodule.findUserById( function (userId, callback) {
  User.findById(userId, callback);
  // callback has the signature, function (err, user) {...}
});
```

Once you have configured this method, you now have access to the user object
that was fetched anywhere in your server app code as `req.user`. For instance:

```javascript
var app = require('express').createServer()

// Configure your app

app.get('/', function (req, res) {
  console.log(req.user);  // FTW!
  res.render('home');
});
```

Moreover, you can access the user in your views as `everyauth.user` or as `user`.

    //- Inside ./views/home.jade
    span.user-id= everyauth.user.name
    #user-id= user.id

## Express Helpers

If you are using express, everyauth comes with some useful dynamic helpers.
To enable them:

```javascript
var express = require('express')
  , everyauth = require('everyauth')
  , app = express.createServer();

everyauth.helpExpress(app);
```

Then, from within your views, you will have access to the following helpers methods
attached to the helper, `everyauth`:

- `everyauth.loggedIn`
- `everyauth.user` - the User document associated with the session
- `everyauth.facebook` - The is equivalent to what is stored at `req.session.auth.facebook`, 
  so you can do things like ...
- `everyauth.facebook.user` - returns the user json provided from the OAuth provider.
- `everyauth.facebook.accessToken` - returns the access_token provided from the OAuth provider
  for authorized API calls on behalf of the user.
- And you also get this pattern for other modules - e.g., `everyauth.twitter.user`, 
  `everyauth.github.user`, etc.

You also get access to the view helper

- `user` - the same as `everyauth.user` above

As an example of how you would use these, consider the following `./views/user.jade` jade template:

    .user-id
      .label User Id
      .value #{user.id}
    .facebook-id
      .label User Facebook Id
      .value #{everyauth.facebook.user.id}

`everyauth` also provides convenience methods on the `ServerRequest` instance `req`. 
From any scope that has access to `req`, you get the following convenience getters and methods:

- `req.loggedIn` - a Boolean getter that tells you if the request is by a logged in user
- `req.user`     - the User document associated with the session
- `req.logout()` - clears the sesion of your auth data

## Configuring a Module

everyauth was built with powerful configuration needs in mind.

Every module comes with a set of parameters that you can configure
directly. To see a list of those parameters on a per module basis, 
with descriptions about what they do, enter the following into the 
node REPL (to access the REPL, just type `node` at the command line)

    > var ea = require('everyauth');
    > ea.facebook.configurable();

For example, you will see that one of the configuration parameters is
`moduleTimeout`, which is described to be `how long to wait per step
before timing out and invoking any timeout callbacks`

Every configuration parameter corresponds to a method of the same name
on the auth module under consideration (i.e., in this case
`ea.facebook`). To create or over-write that parameter, just
call that method with the new value as the argument:

```javascript
ea.facebook
  .moduleTimeout( 4000 ); // Wait 4 seconds before timing out any step
                          // involved in the facebook auth process
```

Configuration parameters can be scalars. But they can be anything. For
example, they can also be functions, too. The facebook module has a 
configurable step named `findOrCreateUser` that is described as 
"STEP FN [findOrCreateUser] function encapsulating the logic for the step
`fetchOAuthUser`.". What this means is that this configures the 
function (i.e., "FN") that encapsulates the logic of this step.

```javascript
ea.facebook
  .findOrCreateUser( function (session, accessToken, extra, oauthUser) {
    // find or create user logic goes here
  });
```

How do we know what arguments the function takes?
We elaborate more about step function configuration in our 
`Introspection` section below.

## Introspection

everyauth provides convenient methods and getters for finding out
about any module.

Show all configurable parameters with their descriptions:

```javascript
everyauth.facebook.configurable();
```

Show the value of a single configurable parameter:

```javascript
// Get the value of the configurable callbackPath parameter
everyauth.facebook.callbackPath(); // => '/auth/facebook/callback'
```

Show the declared routes (pretty printed):

```javascript
everyauth.facebook.routes;
```

Show the steps initiated by a given route:

```javascript
everyauth.facebook.route.get.entryPath.steps; 
everyauth.facebook.route.get.callbackPath.steps;
```

Sometimes you need to set up additional steps for a given auth
module, by defining that step in your app. For example, the
set of steps triggered when someone requests the facebook
module's `callbackPath` contains a step that you must define
in your app. To see what that step is, you can introspect
the `callbackPath` route with the facebook module.

```javascript
everyauth.facebook.route.get.callbackPath.steps.incomplete;
// => [ { name: 'findOrCreateUser',
//        error: 'is missing: its function' } ]
```

This tells you that you must define the function that defines the
logic for the `findOrCreateUser` step. To see what the function 
signature looks like for this step:

```javascript
var matchingStep =
everyauth.facebook.route.get.callbackPath.steps.filter( function (step) {
  return step.name === 'findOrCreateUser';
})[0];
// { name: 'findOrCreateUser',
//   accepts: [ 'session', 'accessToken', 'extra', 'oauthUser' ],
//   promises: [ 'user' ] }
```

This tells you that the function should take the following 4 arguments:

```javascript
function (session, accessToken, extra, oauthUser) {
  ...
}
```

And that the function should return a `user` that is a user object or
a Promise that promises a user object.

```javascript
// For synchronous lookup situations, you can return a user
function (session, accessToken, extra, oauthUser) {
  ...
  return { id: 'some user id', username: 'some user name' };
}

// OR

// For asynchronous lookup situations, you must return a Promise that
// will be fulfilled with a user later on
function (session, accessToken, extra, oauthUser) {
  var promise = this.Promise();
  asyncFindUser( function (err, user) {
    if (err) return promise.fail(err);
    promise.fulfill(user);
  });
  return promise;
}
```

You add this function as the block for the step `findOrCreateUser` just like
you configure any other configurable parameter in your auth module:

```javascript
everyauth.facebook
  .findOrCreateUser( function (session, accessToken, extra, oauthUser) {
    // Logic goes here
  });
```

There are also several other introspection tools at your disposal:

For example, to show the submodules of an auth module by name:

```javascript
everyauth.oauth2.submodules;
```

Other introspection tools to describe (explanations coming soon):

- *Invalid Steps*
    
    ```javascript    
    everyauth.facebook.routes.get.callbackPath.steps.invalid
    ```

## Debugging

### Debugging - Logging Module Steps

To turn on debugging:

```javascript
everyauth.debug = true;
```

Each everyauth auth strategy module is composed of steps. As each step begins and ends, everyauth will print out to the console the beginning and end of each step. So by turning on the debug flag, you get insight into what step everyauth is executing at any time.

### Debugging - Configuring Error Handling

By default, all modules handle errors by throwing them. That said, `everyauth` allows
you to over-ride this behavior.

You can configure error handling at the module and step level. To handle *all*
errors in the same manner across all auth modules that you use, do the following.

```javascript
everyauth.everymodule.moduleErrback( function (err) {
  // Do something with the err -- e.g., log it, throw it
});
```

You can also configure your error handling on a per module basis. So, for example, if
you want to handle errors during the Facebook module differently than in other modules:


```javascript
everyauth.facebook.moduleErrback( function (err) {
  // Do something with the err -- e.g., log it, throw it
});
```

### Debugging - Setting Timeouts

By default, every module has 10 seconds to complete each step. If a step takes longer than 10 seconds to complete, then everyauth will pass a timeout error to your configured error handler (see section "Configure Error Handling" above).

If you would like to increase or decrease the timeout period across all modules, you can do so via:

```javascript
everyauth.everymodule.moduleTimeout(2000); // Wait 2 seconds per step instead before timing out
```

You can eliminate the timeout altogether by configuring your timeouts to -1:

```javascript
everyauth.everymodule.moduleTimeout(-1);
```

You can also configure the timeout period on a per module basis. For example, the following will result in the facebook module having 3 seconds to complete each step before timing out; all other modules will have the default 10 seconds per step before timing out.

```javascript
everyauth.facebook.moduleTimeout(3000); // Wait 3 seconds
```

## Modules and Projects that use everyauth

Currently, the following module uses everyauth. If you are using everyauth
in a project, app, or module, get in touch to get added to the list below:

- [mongoose-auth](https://github.com/bnoguchi/mongoose-auth) Authorization plugin
  for use with the node.js MongoDB orm.

---
### Author
Brian Noguchi

### Credits

Thanks to the following contributors for the following modules:

- [RocketLabs Development](https://github.com/rocketlabsdev) for contributing
  - OpenId
  - Google Hybrid
- [Andrew Mee](https://github.com/starfishmod)
  - OpenId
- [Alfred Nerstu](https://github.com/alfrednerstu)
  - Readability
- [Torgeir](https://github.com/torgeir)
  - DropBox
- [slickplaid](https://github.com/slickplaid)
  - Justin.tv
  - Vimeo
- [Andrew Kramolisch](https://github.com/andykram)
  - Gowalla

### MIT License
Copyright (c) 2011 by Brian Noguchi

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

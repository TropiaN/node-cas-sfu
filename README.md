# node-cas-sfu
node-cas-sfu is a [CAS](http://www.jasig.org/cas) client for [Node.js](http://nodejs.org), tailored to work with [Simon Fraser University's](http://www.sfu.ca/itservices/publishing/publish_howto/enhanced_web_publishing/cas.html) CAS implementation.

# Features
node-cas-sfu supports CAS version 2 and *should* work with a vanilla CAS installation, though this has not been tested. It also supports SFU-specific extensions to CAS, such as:
* the allow string
* using non-SFU "Apache" accounts

# Usage
    var CAS = require('cas');
    var cas = new CAS({
        serverBase: 'http://www.sfu.ca/myapp',  // REQUIRED; the base URL for your application
        allow: '!i-cat',                        // OPTIONAL; defaults to allow=sfu. See http://i.sfu.ca/MWkAlX for a full list of allow options
        userObject: 'AUTH_USER'                 // OPTIONAL; object where the CAS response will be stored. Defaults to 'authenticatedUser'
    });

    // validate a ticket:
    cas.validate(req, ticket, function(err, loggedIn, casResponse) {
        if (loggedIn) {
            console.log("Hello, you are logged in as %s", casResponse.user);
        } else {
            console.log("You are not logged in.");
        }
    });

## Usage in Connect/Express
node-cas-sfu exposes a getMiddleware function to provide Express middleware:

    var cas = require('cas');
    var casauth = cas.getMiddleware({
        serverBase: 'http://www.sfu.ca/myapp',
        allow: '!i-cat',
        userObject: 'AUTH_USER'
    });

    var loggedin = function(req, res, next) {
        if ((req.session && req.session.AUTH_USER) || req.AUTH_USER) {
            // user has an express session and is logged in
            next();
            return;
        }
        // else, redirect to the login route
        res.redirect('/login');
    };

    app.get('/secretstuff', loggedin, function(req, res) {
        res.send('Hello, ' + req.session.AUTH_USER.user);       // Hello, kipling
    });

    app.get('/login', casauth, function(req, res) {
        req.session.AUTH_USER.logindate = new Date();
        res.redirect('/secretstuff');
    });

## Using non-SFU (aka Apache) accounts
SFU's implementation of CAS allows users to authenticate with made-up, non-SFU accounts. These are often referred to as "Apache accounts" as they are most commonly used in Apache .htpasswd files via the [mod_auth_cas](http://www.sfu.ca/itservices/publishing/publish_howto/enhanced_web_publishing/cas/apache_module.html) Apache module. node-cas-sfu also supports Apache accounts; you can use them by setting `allow=apache` and including an `apacheUsers` object containing username & password hash pairs:

    var CAS = require('cas');
    var cas = new CAS({
        serverBase: 'http://www.sfu.ca/myapp',
        allow: 'apache',
        userObject: 'AUTH_USER',
        apacheUsers: {"myfakeuser": "ubjQPM.hh9Qj2"}
    });

Passwords can be any of UNIX crypt, SHA1, Apache MD5 or even plain text (but really, don't do plain text). node-cas-sfu uses the [pass](https://github.com/andris9/pass) module to validate hashes.


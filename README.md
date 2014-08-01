### hapi-auth-bearer

[![Build Status](https://travis-ci.org/j/hapi-auth-bearer.png?branch=master)](https://travis-ci.org/j/hapi-auth-bearer)

#### Bearer authentication

This scheme requires the following options:

- `validateFunc` - Function with signature `function(secretOrToken, callback)` where:
    - `secretOrToken` - the `secret` if option `base64: true` is set, otherwise the raw token value is passed in.
    - `callback` - the callback function with signature `function(err, credentials)` where:
        - `err` - an internal error.
        - `credentials` - a credentials object that gets passed back to the application in `request.auth.credentials`.
          Return `null` or `undefined` to when the credentials are unknown (and not an error).

- `base64` - Boolean value (defaults to `false` aka just accepts a raw token value).  This gives you the ability to pass
 back a base64 encoded authorization header: base64(SECRET:TOKEN)
    - i.e.) Bearer NTJlYjRmZmRmM2M3MjNmZjA1MTEwYmYxOjk5ZWQyZjdmMWRiNjBiZDBlNGY1ZjQ4ZjRhMWVhNWVjMmE4NzU2ZmU=


#####  Using Token
```javascript
var Hapi = require('hapi');
var server = new Hapi.Server(8088);
var bearer = require('hapi-auth-bearer');

var credentials = {
    someSuperSecureToken: {
        user: {
            "Name": "Username",
            "token": "someSuperSecureToken"
        }
    }
};

var validateFunc = function (token, callback) {
    console.log(token)
    console.log(credentials[token])
    var result = credentials[token]
    if (!!result) {
        callback(null, result['user']);
    } else {
        callback(null, null);
    }
};

server.pack.register(bearer, function (err) {
    if (err) {
        console.log(err);
    }
    server.auth.strategy('bearer', 'bearer', {
        validateFunc: validateFunc
    });
    server.route([
        {
            method: ['GET', 'POST'],
            path: '/login',
            config: {
                handler: function (request, reply) {
                    reply("Hello")
                },
                auth: {
                    strategy: 'bearer'
                },
                plugins: {
                    'hapi-auth-bearer': {
                        redirectTo: false
                    }
                }
            }
        }
    ]);
    server.start(function () {
        console.log('Server running at:', server.info.uri);
    })
});

```
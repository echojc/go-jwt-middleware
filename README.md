# GO JWT Middleware

A middleware that will check that a [JWT](http://jwt.io/) is sent in the
`Authorization` header and will then set the content of the JWT into the `user`
variable of the request context.

This is a fork of
[auth0/go-jwt-middleware](https://github.com/auth0/go-jwt-middleware) that
removes all external dependencies except for
[dgrijalva/jwt-go](https://github.com/dgrijalva/jwt-go), which is vendored.

## Key Features

* Ability to **check the `Authorization` header for a JWT**
* **Decode the JWT** and set the content of it to the request context

## Installing

```bash
# install globally
go get github.com/echojc/go-jwt-middleware

# vendor with gvt
gvt fetch github.com/echojc/go-jwt-middleware
```

## Using it

You can use `jwtmiddleware` with default `net/http` as follows.

```go
package main

import (
  "fmt"
  "net/http"

  "github.com/dgrijalva/jwt-go"
  "github.com/echojc/go-jwt-middleware"
)

func main() {
  myHandler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    user := r.Context().Value("user").(*jwt.Token)
    fmt.Fprintf(w, "This is an authenticated request")
    fmt.Fprintf(w, "Claim content:\n")
    for k, v := range user.Claims {
      fmt.Fprintf(w, "%s :\t%#v\n", k, v)
    }
  })

  jwtMiddleware := jwtmiddleware.New(jwtmiddleware.Options{
    ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
      return []byte("My Secret"), nil
    },
    // When set, the middleware verifies that tokens are signed with the specific signing algorithm
    // If the signing method is not constant the ValidationKeyGetter callback can be used to implement additional checks
    // Important to avoid security issues described here: https://auth0.com/blog/2015/03/31/critical-vulnerabilities-in-json-web-token-libraries/
    SigningMethod: jwt.SigningMethodHS256,
  })

  app := jwtMiddleware.Handler(myHandler)
  http.ListenAndServe("0.0.0.0:3000", app)
}
```

## Options

```go
type Options struct {
  // The function that will return the Key to validate the JWT.
  // It can be either a shared secret or a public key.
  // Default value: nil
  ValidationKeyGetter jwt.Keyfunc
  // The name of the property in the request where the user information
  // from the JWT will be stored.
  // Default value: "user"
  UserProperty string
  // The function that will be called when there's an error validating the token
  // Default value: DefaultErrorHandler (returns 401 Unauthorized)
  ErrorHandler ErrorHandler
  // A boolean indicating if the credentials are required or not
  // Default value: false
  CredentialsOptional bool
  // A function that extracts the token from the request
  // Default: DefaultTokenExtractor (extracts from Authorization header as bearer token)
  Extractor TokenExtractor
  // Debug flag turns on debugging output
  // Default: false
  Debug bool
  // When set, all requests with the OPTIONS method will use authentication
  // Default: false
  EnableAuthOnOptions bool,
  // When set, the middelware verifies that tokens are signed with the specific signing algorithm
  // If the signing method is not constant the ValidationKeyGetter callback can be used to implement additional checks
  // Important to avoid security issues described here: https://auth0.com/blog/2015/03/31/critical-vulnerabilities-in-json-web-token-libraries/
  // Default: nil
  SigningMethod jwt.SigningMethod
}
```

### Token Extraction

The default value for the `Extractor` option is the `DefaultTokenExtractor`
function which assumes that the JWT will be provided as a bearer token
in an `Authorization` header, i.e.,

```
Authorization: bearer {token}
```

To extract the token from a different header, you can use the `FromHeader`
function, e.g.,

```go
jwtmiddleware.New(jwtmiddleware.Options{
  Extractor: jwtmiddleware.FromHeader("X-My-Header"),
})
```

To extract the token from a query string parameter, you can use the
`FromParameter` function, e.g.,

```go
jwtmiddleware.New(jwtmiddleware.Options{
  Extractor: jwtmiddleware.FromParameter("auth_code"),
})
```

In this case, the `FromParameter` function will look for a JWT in the
`auth_code` query parameter.

Or, if you want to allow both, you can use the `FromFirst` function to
try and extract the token first in one way and then in one or more
other ways, e.g.,

```go
jwtmiddleware.New(jwtmiddleware.Options{
  Extractor: jwtmiddleware.FromFirst(jwtmiddleware.DefaultTokenExtractor,
                                     jwtmiddleware.FromParameter("auth_code")),
})
```

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amont others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a free Auth0 Account

1. Go to [Auth0](https://auth0.com) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Contributors

* [Auth0](https://auth0.com/)
* Jonathan Chow

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for more info.

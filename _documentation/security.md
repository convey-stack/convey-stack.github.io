---
title: Security
description: JWT authentication.
keywords:
order: 11
comments: false

hero:
    title: Security
    text: JWT authentication.
---

## JWT
Adds the integration with [JWT](https://jwt.io) using an available authentication middleware and system components to validate and grant the access tokens. 

## Installation
`dotnet add package Convey.Auth`

## Dependencies

* [Convey](https://www.nuget.org/packages/Convey)
* [Convey.Persistence.Redis](https://www.nuget.org/packages/Convey.Persistence.Redis)
  
## Usage

Extend `IConveyBuilder` with `AddJwt()` that will register the required services.

```csharp
public static IConveyBuilder RegisterConvey(this IConveyBuilder builder)
{
    builder.AddJwt()
    // Other services.

    return builder;
}
```

Then, invoke `UseAuthentication()` extension from `IApplicationBuilder`.

```csharp
public static IApplicationBuilder UseConvey(this IApplicationBuilder app)
{
    app.UseAuthentication();
    // Other services.

    return app;
}
```

Creating the access tokens can be done by using `IJwtHandler` interface.

```csharp
public class UserService
{
    private readonly IJwtHandler _jwtHandler;

    public UserService(IJwtHandler jwtHandler)
    {
        _jwtHandler = jwtHandler;
    }
    
    public async Task<string> SignInAsync(string email, string password)
    {
        var user = ... //Fetch user from a custom database
        ValidateCredentials(user, password); //Validate the credentials

        //Generate the token with an optional role and other claims
        var token = _jwtHandler.CreateToken(user.Id, user.Role, user.Claims); 

        return token.AccessToken;
    }
}
```

To blacklist and deactivate the access tokens, use `IAccessTokenService` and invoke `UseAccessTokenValidator()` extension. Blacklisted tokens are kept in `Redis` cache for the period of their expiry.

## Options
* `secretKey` - a secret key used to create the access tokens.
* `issuer` - a party signing the tokens.
* `expiryMinutes` - how long the token will remain valid.
* `validateLifetime` - if `true` then the lifetime defined in `expiryMinutes` will be validated.
* `validAudience` - an audience that can use the access tokens.
* `validateAudience` - if `true` then the audience defined in `validAudience` will be validated.

### appsettings.json

```js
"vault": {
  "secretKey": "secret,
  "issuer": "identity-service",
  "expiryMinutes": 5,
  "validateLifetime": true,
  "validAudience": "audience",
  "validateAudience": true,
}
```
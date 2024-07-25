
# Retrieving user information from AWS Amplify authentication
**Daniel Dantas** - **Dec 28, 2019**

This is a guide to three methods of retrieving user information from AWS Amplify authentication: `Auth.currentSession()`, `Auth.currentUserInfo`, and `Auth.currentAuthenticatedUser()`

There is official documentation in three places:

1.  [AWS Amplify: Working with the API](https://aws-amplify.github.io/docs/js/authentication#working-with-the-api)
2.  [AWS Amplify: AuthClass API](https://aws-amplify.github.io/amplify-js/api/classes/authclass.html)
3.  [AWS Cognito Developer Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/)

But it wasn’t clear to me what the returned object of these three methods looked like, and how these methods related to each other, so this is an attempt to synthesize these three documentation sources and include some experimentation of my own.

For the below examples, I used the default `amplify add auth` settings, which use [AWS Cognito User Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html). Your results will vary if you use different `amplify add auth` settings.

## Auth.currentSession()

Described in the [AWS Amplify: Retrieve Current Session](https://aws-amplify.github.io/docs/js/authentication#retrieve-current-session) documentation, the `Auth.currentSession()` method retrieves the access, id, and refresh tokens.  
The contents of these three tokens are described in the [AWS Cognito: Using Tokens](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html) documentation.  
For the default `amplify add auth` settings, the object returned by the `Auth.currentSession()` method has the following form:

```json
{  
  "accessToken": {  
    "jwtToken": "XXXX",  
    "payload": {  
      "auth_time": "XXXX",  
      "client_id": "XXXX",  
      "event_id": "XXXX-XXXX-XXXX-XXXX-XXXX",  
      "exp": "XXXX",  
      "iat": "XXXX",  
      "iss": "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_XXXX",  
      "jti": "XXXX-XXXX-XXXX-XXXX-XXXX",  
      "scope": "aws.cognito.signin.user.admin",  
      "sub": "INTERNAL USERID: XXXX-XXXX-XXXX-XXXX-XXXX",  
      "token_use": "access",  
      "username": "MY USERNAME"  
    }  
  },  
  "clockDrift": 0,  
  "idToken": {  
    "jwtToken": "XXXX",  
    "payload": {  
      "aud": "XXXX",  
      "auth_time": "XXXX",  
      "cognito:username": "MY USERNAME",  
      "email": "MY EMAIL ADDRESS",  
      "email_verified": true,  
      "event_id": "XXXX-XXXX-XXXX-XXXX-XXXX",  
      "exp": "XXXX",  
      "iat": "XXXX",  
      "iss": "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_XXXX",  
      "sub": "INTERNAL USERID: XXXX-XXXX-XXXX-XXXX-XXXX",  
      "token_use": "id"  
    }  
  },  
  "refreshToken": {  
    "token": "XXXX"  
  }  
}
```

## Auth.currentUserInfo()

The `Auth.currentUserInfo()` method retrieves the [AWS Cognito User Attributes](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-attributes.html) for the current user.  
For the default `amplify add auth` settings, the object returned by the `Auth.currentUserInfo()` method has the following form:

```json
{  
  "attributes": {  
    "email": "MY EMAIL ADDRESS",  
    "email_verified": true,  
    "sub": "INTERNAL USERID: XXXX-XXXX-XXXX-XXXX-XXXX"  
  },  
  "id": "us-east-1:XXXX",  
  "username": "MY USERNAME"  
}
```

## Auth.currentAuthenticatedUser()

Described in the [AWS Amplify: Retrieve Current Authenticated User](https://aws-amplify.github.io/docs/js/authentication#retrieve-current-authenticated-user) documentation, the `Auth.currentAuthenticatedUser()` method returns a combination of the result of the `Auth.currentUserInfo()` method, the result of the `Auth.currentSession()`  method, and some extra information.  
For the default `amplify add auth` settings, the `Auth.currentUserPoolUser()` method returns the same object as the `Auth.currentAuthenticatedUser()` method, and that object has the following form:

```json
{  
  "Session": null,  
  "attributes": {  
    "... SAME AS AUTH.CURRENTUSERINFO()"
  },  
  "authenticationFlowType": "USER_SRP_AUTH",  
  "client": {  
    "endpoint": "https://cognito-idp.us-east-1.amazonaws.com/",  
    "userAgent": "aws-amplify/0.1.x react-native"  
  },  
  "keyPrefix": "CognitoIdentityServiceProvider.XXXX",  
  "pool": {  
    "advancedSecurityDataCollectionFlag": true,  
    "client": {  
      "endpoint": "https://cognito-idp.us-east-1.amazonaws.com/",  
      "userAgent": "aws-amplify/0.1.x react-native"  
    },  
    "clientId": "XXXX",  
    "storage": "Function MemoryStorage",  
    "userPoolId": "us-east-1_XXXX"  
  },  
  "preferredMFA": "NOMFA",  
  "signInUserSession": {  
     "... THE ACCESS, ID & REFRESH TOKENS OF AUTH.CURRENTSESSION()"
 },  
  "storage": "Function MemoryStorage",  
  "userDataKey": "CognitoIdentityServiceProvider.XXXX.XXXX.userData",  
  "username": "MY USERNAME"  
}
```

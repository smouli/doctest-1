
# OpenID Connect Minio Deployment Guide

To authenticate applications using OpenID Connect register your application with the Identity Provider(wso2).

## How OpenID works?

![image](https://user-images.githubusercontent.com/22103395/41444342-662c7a42-6ff7-11e8-93aa-75bce207e6cd.png)

#### 1. Client request for access_token from wso2 by sending client id and client secret.

Request
```
curl -v -X POST -H "Authorization: Basic <base64 encoded client id:client secret value>" -k -d "grant_type=client_credentials" -H "Content-Type:application/x-www-form-urlencoded" https://localhost:9443/oauth2/token
```

#### 2. wso2 returns access_token.

Response
```
{"token_type":"Bearer","expires_in":2061,"access_token":"ca19a540f544777860e44e75f605d927"}

```

Client credentials grant in wso2 don't support refresh token. To rotate temporary credentials, Minio would provide an API to request new access key, before the current access key expires. AWS uses a similar process documented [here](https://aws.amazon.com/blogs/security/how-to-rotate-access-keys-for-iam-users/).

#### 3. Client request for temporary credentials by making AssumeRoleWithClientGrant request to Minio with the access_token.

#### 4. Minio verify access_token by making a POST request to OAuth Introspection Endpoint using username and password.

Request 
```
curl -k -u <USERNAME>:<PASSWORD> -H 'Content-Type: application/x-www-form-urlencoded' -X POST --data 'token=<ACCESS_TOKEN>' https://localhost:9443/oauth2/introspect
```

Here USERNAME can be of any user with "/permission/admin/manage/identity/applicationmgt/view" permissions. For more information, refer to the documentation [here](https://docs.wso2.com/display/IS530/Invoke+the+OAuth+Introspection+Endpoint).

#### 5. wso2 returns a response whether the token is valid or not. If valid, responds with the expiry time of access_token


Response
```
{"exp":1464161608,"username":"admin@carbon.super","active":true,"token_type":"Bearer","client_id":"rgfKVdnMQnJSSr_pKFTxj3apiwYa","iat":1464158008}
```

#### 6. If access_token is valid, Minio returns temporary credentials comprising of Access key, Secret key and Session token and expiry time.

  

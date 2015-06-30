# Authorization Server Client Documentation #
---------------------------------------------

### Introduction ###
The Cerner Authorization Server currently supports OAuth 2.0 [1] SMART [2, 3] on FHIR launch workflows. As a client, it will require interaction between the client (you), the user, the authorization server, a SMART Launch server and a FHIR resource server.

### Registration ###
In order for your client application to utilize any FHIR protected resources, you must first register your client application. To do this, you can contact your Cerner representative with the following information, the following fields are **required**.

* Name
* Redirect/Callback URI
* Email address

You may also provide the following **optional** fields.

* Logo URI 
* SMART Launch Server URI

Once approved, a **client identifier** will be provided to you for use with the Cerner Authorization Server. As a registered client, Cerner tenants may then ask for your client to be enabled, which is necessary in order to gain access to their protected FHIR resources.

### Supported Scopes ###
The Cerner Authorization Server supports many, but not all of the SMART or OAuth scopes. The following scopes are supported.

* online_access
* launch

### Requesting an authorization code ###
Cerner currently supports the SMART launch workflow from within an EHR, such as Cerner Millennium's Powerchart. This allows for current context information (patient info, encounter info, user info, etc.) to be provided to the client application upon launching. Below is a flowchart of the EHR SMART luanch workflow.

![alt text](http://www.websequencediagrams.com/cgi-bin/cdraw?lz=dGl0bGUgRUhSIEFwcCBMYXVuY2ggRmxvdwoKTm90ZSAgbGVmdCBvZiBFSFI6IFVzZXIgbAAgBWVzIGFwcApFSFItPj5BcHA6IFJlZGlyZWN0IHRvIGFwcDoAIgYAQQZyaWdoAEIFACMHcXVlc3QgYXV0aG9yaXphdGlvbgpBcHAtPj4AYwUAPwxlaHI6ACEIZQCBARRPbiBhcHByb3ZhbABzHHIAgRgHX3VyaT9jb2RlPTEyMyYuLi4AcwYAgVsFUE9TVCAvdG9rZW4AGgkAggIGAIF6DUNyZWF0ZSAAIwU6XG4ge1xuYWNjZXNzXwA2BT1zZWNyZXQtAEMFLXh5eiZcbnBhdGllbnQ9NDU2JlxuZXhwaXJlc19pbjogMzYwMFxuLi4uXG59Cn0AgksGAIJLBVsATgYAYQYgcmVzcG9uc2VdAIMSBwCCRA5BACUGAF8HIGRhdGFcbnZpYSBGSElSIEFQSQCBWgtHRVQgL2ZoaXIvUACBDwYvNDU2XG5BAIMBDDogQmVhcmVyIACBOxAAg2wFAIEaB3tyZXNvdXJjZVR5cGU6ICIASAciLCAiYmlydGhEYXRlIjogLi4ufQo&s=default "SMART launch diagram")

Once a launch context is received by your client from the SMART launch server, it must be sent to the authorization server so that an authorization code may be requested. To request an authorization code, to turn in for an access code, you will need to issue a **GET** request to the authorization servers' authorize endpoint. At thsi point the authorization server will redirect the user to authenticate if they are not already authenticated. Your application should use the systems browser for performing this exchange rather than an embedded browser. Using embedded browsers prevents single sign-on and authentication may fail for certain organizations who implement third party authentication systems.

The authorize endpoints are tenant specific, which means you will need to know the specific tenant ID of the tenants whose data you are going to request access to as well as the following **required** query parameters.

* response_type
* client_id
* launch
* scope - Note: launch **must** be one of the scopes if using a launch context

It is also recommended that you provide the following query parameters.

* state - to prevent cross-site requst forgery [4] attacks
* redirect_uri - note: The redirect_uri ***must*** match what was originally registered

```
https://authorization.stagingcerner.com/oauth2/{TENANT_ID}/authorize?response_type=code&client_id={YOUR_CLIENT_ID}&state=12345&redirect_uri=https%3A%2F%2Ftest.com%2Fcb
```

If the user is not currently have an active session [4], the Authorization Server will redirect the user to the identity provider for the tenant in order to authenticate. 

If successful, the authorization server will redirect the user-agent back to your client apps redirect_uri with a **code** query paramter. This is your authorization code that you will exchange for an access token.

```
https://test.com/cb?code=1234-567890ab-cdef
```

### Requesting an access token ###
Once you have your authorization code, you can now make a back-channel POST to the authorization servers token endpoint to request an access token. Following the OAuth 2.0 [1] spec, the authorization server accepts a content type of **application/x-www-form-urlencoded** POST from your client application with the following information.

* grant_type
* code
* client_id
* redirect_uri (only if provided when the authorization code was requested)

```
https://authorization.stagingcerner.com/oauth2/{TENANT_ID}/token
```

```
grant_type=authorization_code&code={AUTHORIZATION_CODE}&client_id={YOUR_CLIENT_ID}&redirect_uri={YOUR CALLBACK URI}
```

If successful, the authorization server will return a **200 OK** response with a content-type of **application/json** [5] similar to the example below.

```
{
   "access_token":"eyJraWQiOiIyMDE1LTA1LTE1VDE4OjI2OjUxLjM0NiIsInR5cCI6IkpXVCIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJLTDAyODA2NSIsInVybjpjb206Y2VybmVyOmF1dGhvcml6YXRpb24uY2xhaW1zIjp7InZlciI6IjEuMCIsInRudCI6IjhlMzNhZTNkLTQ4NzItNGQyZi1hMDU5LTVmMjMwN2ZjYmYzYiJ9LCJhenAiOiIyIiwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0OjgwODBcL2F1dGhvcml6YXRpb25cLzhlMzNhZTNkLTQ4NzItNGQyZi1hMDU5LTVmMjMwN2ZjYmYzYlwvdG9rZW4iLCJleHAiOjE0MzE3MTUxNDcsImlhdCI6MTQzMTcxNDU0NywianRpIjoiZmIzZDMwOGQtZmQ0Yy00MTk5LTlmZDQtZTU2ZmEwM2Y3ZTViIn0.Fv_6-LtIFzvf7DHleH-rXsjnaEMFgTHRyok4vJ8hkml5FQtnKejNSwECvqdex7hz6VyclcMy67D_bcafNZjNb",
   "token_type":"Bearer",
   "expires_in":600,
   "refresh_token":"1234-567890ab-cdef"
}
```

The bearer access token returned from the authorization server is a JSON Web Token (JWT) [6]. The JWT is what you provide to the protected FHIR resource. If a refresh token was also requested, it will be returned as well. Access tokens are good for 10 minutes and it is recommended refreshing it before use if less than 5 minutes remain before it expires.  

### Using an access token ###
In order to use an access token, you need to provide the access token recieved from the authorization server to the FHIR protected resource. The following is a non-normative example of the usage of the token to access a protected RESTful web service on a resource server, this needs to be added as an authorization HTTP header.

```
Authorization: Bearer {ACCESS_TOKEN}
```

If the access token is valid, the FHIR protected resource will grant your client application access to it's protected resources.

### Using a Refresh Token ###
The authorization server has support for **online access** refresh tokens. In order to use the refresh token, the users session **must** remain active. To request a refresh token, you must add **online_access** to the scope query parameter when requesting an authorization code. If this scope is not added, a refresh token will not be returned. Following the OAuth 2.0 [1] spec, the authorization server accepts a content type of **application/x-www-form-urlencoded** POST from your client application with the following information.

* grant_type
* refresh_token

```
grant_type=refresh_token&refresh_token={REFRESH_TOKEN}
```

If successful, the authorization server will return a **200 OK** response with a content-type of **application/json** [5] similar to the example below. 

```
{
   "access_token":"eyJraWQiOiIyMDE1LTA1LTE1VDE4OjI2OjUxLjM0NiIsInR5cCI6IkpXVCIsImFsZyI6IkVTMjU2In0.eyJzdWIiOiJLTDAyODA2NSIsInVybjpjb206Y2VybmVyOmF1dGhvcml6YXRpb24uY2xhaW1zIjp7InZlciI6IjEuMCIsInRudCI6IjhlMzNhZTNkLTQ4NzItNGQyZi1hMDU5LTVmMjMwN2ZjYmYzYiJ9LCJhenAiOiIyIiwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0OjgwODBcL2F1dGhvcml6YXRpb25cLzhlMzNhZTNkLTQ4NzItNGQyZi1hMDU5LTVmMjMwN2ZjYmYzYlwvdG9rZW4iLCJleHAiOjE0MzE3MTUxNDcsImlhdCI6MTQzMTcxNDU0NywianRpIjoiZmIzZDMwOGQtZmQ0Yy00MTk5LTlmZDQtZTU2ZmEwM2Y3ZTViIn0.Fv_6-LtIFzvf7DHleH-rXsjnaEMFgTHRyok4vJ8hkml5FQtnKejNSwECvqdex7hz6VyclcMy67D_bcafNZjNb",
   "token_type":"Bearer",
   "expires_in":600
}
```

The refresh token issued is good while the users session is still valid. 

##### References #####
[1] https://tools.ietf.org/html/rfc6749  
[2] http://smarthealthit.org/smart-on-fhir/  
[3] http://docs.smarthealthit.org/  
[4] https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)  
[5] http://json.org/  
[6] https://tools.ietf.org/html//rfc7519

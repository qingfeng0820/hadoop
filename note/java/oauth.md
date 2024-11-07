## Response Code (授权码模式)
* application server: a.com, auth provider server: b.com

1. Ask client to redirect to auth provider server for login and granting auth.
   ```
   https://b.com/oauth/authorize?
    response_type=code&
    client_id=CLIENT_ID&
    redirect_uri=CALLBACK_URL&
    scope=read
   ```
2. After login and grant auth from auth provider server, auth provider server will callback to application server
   * the callback url specified by step 1 via `redirect_uri=CALLBACK_URL`
   * redirect_url: can be used for verification -> check it's same as the registered redirect_uri
   ```
   https://a.com/callback?code=AUTHORIZATION_CODE
   ```
3. Application server gets the AUTHORIZATION_CODE, then it requests the access token (in application server backend) from auth provider server
   * redirect_url: just used for verification? check it's same as the registered redirect_uri
   ```
   https://b.com/oauth/token?
    client_id=CLIENT_ID&
    client_secret=CLIENT_SECRET&
    grant_type=authorization_code&
    code=AUTHORIZATION_CODE&
    redirect_uri=CALLBACK_URL
   ```
4. auth provider server will check the CLIENT_ID, CLIENT_SECRET and AUTHORIZATION_CODE, then responses the access token.
   * access_token can be jwt format token
   ```
   {    
    "access_token":"ACCESS_TOKEN",
    "token_type":"bearer",
    "expires_in":2592000,
    "refresh_token":"REFRESH_TOKEN",
    "scope":"read",
    "uid":100101,
    "info":{...}
    }
   ```
   
5. Application server can use refresh_token to get a new access_token (and refresh_token) before the access_token expired.
   ```
   https://b.com/oauth/token?
    grant_type=refresh_token&
    client_id=CLIENT_ID&
    client_secret=CLIENT_SECRET&
    refresh_token=REFRESH_TOKEN
   ```
   
6.  auth provider server will check the CLIENT_ID, CLIENT_SECRET and REFRESH_TOKEN, then responses the new access token via the CALLBACK_URL in step 3.
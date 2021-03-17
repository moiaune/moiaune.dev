---
title: "Authenticate With Azure Ad"
date: 2021-03-17T20:44:57+01:00
draft: true
---

In this post we'll go through how to configure and add Azure AD authentication to a website without any third party libraries. We will be using the authorization code flow, which will require the use to login as him/herself.

## Setting up an App Registration in Azure

Our first step is to setup a new App Registration in Azure. This is where we will be able to say which users have access to use this login method, and it also kinda works as a proxy between our app and Azure AD itself. The only thing you really need to configure here is to give it a name, set the callback URL (use `http://localhost:8080/callback.php`) and generate a new secret.

TODO: Add more detailed step for setting up a new App Registration.

## Acquire a token

There are atleast two ways to acquire an access token for authenticating with Azure REST API, Microsoft Graph API and so on. Using the **authorization code** flow or the **client credentials** flow. Here I'm going to show you the **authorization code** flow which is used to authenticate as the user. This is a two step process.

First, you need to get an authorize code, by redirecting your users to a specially crafted URL at Microsoft. This will give the user the option login in. Upon success, Microsoft will redirect the user back to the callback URL, defined for our App Registration, along with an authorization code.

Then, we need to use the authorization code and do a POST request to another Microsoft URL which will respond with our access token upon success. This access token can be passed in the HTTP header as `Authorization: Bearer <token>` for further requests to the Azure REST API.

So let's get going and craft our authorization code URL.

### Request an authorization code

So let's start by creating our project folder.

```bash
mkdir aad-authorization-code-flow
cd aad-authorization-code-flow
touch index.php
```

Our authorize URL will look something like this in the end.

```plaintext
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=<APP_REGISTRATION_CLIENT_ID>
&response_type=code
&redirect_uri=<CALLBACK_URL>
&response_mode=query
&scope=<SCOPE>
&state=<STATE>
```

1. The `tenant` can either be: `common`, `organizations`, `consumers` or your tenant id if you only want to allow users from your tenant.
2. `client_id` is the `Application ID` for your App Registration that you setup earlier.
3. `response_type` must be `code` when using the authorization flow.
4. `redirect_uri` must match the URL that you setup as callback URL in your App Registration. The URL must be urlencoded.
5. `response_mode=query` is the method that Microsoft will use when returning with the authorization code. `query` meaning it will be in the URL as `?code=<authorization_code>`.
6. `scope` is what permissions your are asking for. In our example will use the `user.read` to be able to get some user information from our logged in user. You can seperate multiple scopes by using spaces. The value must be URL encoded.
7. `state` is a value included in the request that will also be returned in the token response. Typically used for preventing XSS attacks.

#### Example

```php
// filename: index.php

<?php

function buildAuthorizeCodeURL($tenantID, $clientID, $callbackURI, $scope) {
    $baseURL = sprintf('https://login.microsoftonline.com/%s/oauth2/v2.0/authorize?', $tenantID);
    $query = array(
        'client_id'     => $clientID,
        'response_type' => 'code',
        'redirect_uri'  => $callbackURI,
        'response_mode' => 'query',
        'scope'         => $scopes,
        'state'         => '',
    );

    return $baseURL . http_build_query($params, '', '&', PHP_QUERY_RFC3986);
}

$authorizeURL = buildAuthorizeCodeURL(
    'XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX',
    'XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX',
    'http://localhost:8080/callback.php',
    'offline_access user.read'
);

?>

<a href="<?= $authorizeURL ?>">Login with Microsoft</a>
```

On line 14 we build out the query params with the built in PHP function `http_build_query`. By default, this function will replace spaces with `+` sign. The more modern approach is to use `%20` to signal a space. There, in the last paramter, we have to specify `PHP_QUERY_RFC3986` which will url encode with `%20` instead of `+`.

When a user clicks the link, he/she will be redirected to Microsoft login where he/her can choose which Microsoft account to login in with. If they are already logged in, they will immediately be redirected back to our `callbackURI` which should log them in, a.k.a Single Sign-On.

So next we need to setup our callback to retrieve the authorization code. Let's make a new file called `callback.php`.

```bash
touch callback.php
```

And then add some code. I'll explain in afterwards.

```php
// filename: callback.php

<?php

if (! isset($_GET['code'])) {
    throw new Exception("Missing query parameter 'code'");

    if (isset($_GET['error'])) {
      // Log something
    }
}

$code = $_GET['code'];
$state = $_GET['state'];

$baseURL     = sprintf('https://login.microsoftonline.com/%s/oauth2/v2.0/token', 'XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX');
$contentType = ;

$postfields = array(
    'client_id'     => 'XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX',
    'client_secret' => 'klsjlkJSIjsiqs7Q8S98sa789SHkajsh',
    'grant_type'    => 'authorization_code',
    'scope'         => 'offline_access user.read',
    'code'          => $code,
    'redirect_uri'  => 'http://localhost:8080/callback.php',
);

$options = array(
    CURLOPT_URL            => $baseURL,
    CURLOPT_POST           => 1,
    CURLOPT_HEADER         => 0,
    CURLOPT_POSTFIELDS     => http_build_query($postfields, '', '&', PHP_QUERY_RFC3986),
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER     => ['Content-Type: application/x-www-form-urlencoded'],
);

$ch = curl_init();
curl_setopt_array($ch, $options);
$response = curl_exec($ch);
curl_close($ch);

$body = json_decode($response);

// $body->access_token
// $body->refresh_token
// $body->expires_in
```
We start by checking if we have recieved the `?code=` query parameter. If its not in the request, the authorization part failed or someone called this file manually. Then we look for the `?error=` query parameter, which will be present in case authorization process failed.

Next, we grab the code and state and start building the `/token` URL which will return a access token on success.

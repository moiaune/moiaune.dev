---
title: "Generate Microsoft Partner Center Refresh Token"
date: 2021-11-03T23:22:03+01:00
tags: ["powershell", "partner-center"]
draft: true
---

Microsoft Partner Center is a portal where you can manage all of your CSP customers and can give you a lot of access and power to do so. Therefor you should naturally have great security on the users that has access to this portal. Like MFA for example.

Hopefully you have MFA enabled on all your Partner Center users, as you should. But MFA does not work great with unattended authentication, like in scripts for example. So how can we then do unattended authentication and automate some of the tasks in Partner Center?

In the Partner Center you can create something they call "Web apps" or "Native apps", which works like a service principal, but they will not give you access to your customers data. For that, you will need to authenticate as a Partner Center user that has either "Admin agent", "Sales agent" or "Helpdesk agent" (depending on access level) in addition to using an Azure service principal. They call this "App + User authentication".

To use the Partner Center SDK or REST API with these permissions, and without having to use MFA all the time, you must create a refresh token. Creating this refresh token is a manual one time job and it will be valid for 90 days. When used it will reset the timer, but if not used for 90 days it will expire. Let's see how we can generate a refresh token. In this example we're going to use Powershell with the Partner Center Powershell module.

## 1. Create an Azure Service Principal

The first step is to create an Azure service principal in the same tenant as your Partner Center.

1. Create a new App registration
2. Generate a secret

## 2. Generate refresh token

When the service principal is in-place your can generate a refresh token by combining the service principal with your Partner Center user credentials.

>NOTE: `$appId`, `$appSecret` and `refreshToken` should be stored safely, like in Azure Key Vault, and fetched from there instead of hardcoded for security reasons.

```powershell
$appId = "" # Service principal app id
$appSecret = ConvertTo-SecureString -String "" -AsPlainText # Service principal secret
$tenantId = "" # Partner Center tenant id
$credential = [PSCredential]::new($appId, $appSecret)

$tokenSplat = @{
    ApplicationId        = $appId
    Credential           = $credential
    Scopes               = "https://api.partnercenter.microsoft.com/user_impersonation"
    ServicePrincipal     = $true
    TenantId             = $tenantId
    UseAuthorizationCode = $true
}

$token = New-PartnerAccessToken @tokenSplat
```

This will open a new tab in your browser and ask you to login. Now you must login with your Partner Center user credentials. When that is done, and if successfull, your refresh token is now stored in `$token` and can be accessed like `$token.RefreshToken`.

## 3. Connect to Partner Center

Now to use this refresh token to authenticate to Partner Center we do this.

```powershell
$connectSplat = @{
    ApplicationId = $appId
    Credential    = $credential
    RefreshToken  = $token.RefreshToken
}

Connect-PartnerCenter @connectSplat
```

You should now be logged in with the same permissions as your Partner Center user.

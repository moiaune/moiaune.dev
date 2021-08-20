---
title: 'Interacting with Azure Key Vault in Go'
date: 2021-08-20T23:18:32+02:00
draft: false
---

Most times when working with API's there some kind of documentation on how to iteract with it. Working with Azure SDK for Go is a different story. There's almost no documentation (except the code itself). At my current job we use Azure a lot and a big part of that is Azure Key Vault. For my latest project I had to fetch some secrets from Key Vault to use in a CLI application, so I had to start digging into the source code to find how to interact with it.

## 1. Authentication

Almost any endpoint in the Azure API requires authentication, so let's start with that. Services in the Azure API, for the most part, use the `autorest/azure/auth` module for handling authentication, but for Key Vault it is a bit different. For Key Vaults we have two modules; one for managing Key Vaults, and one for working with the data.

- Management: github.com/Azure/go-autorest/autorest/azure/auth
- Data: github.com/Azure/azure-sdk-for-go/services/keyvault/auth

This is very important, because if you type "auth.NewAuthorizerFromCLI" in your editor and have auto imports on, it will most likely use the autorest module, which will give you an error when working with the data inside Key Vault.

Another important concept to know, is that the Azure SDK for Go use something called "Authorizer". As we'll see in this sample, we need to initiate an "authorizer" from one of the auth modules, and then pass that into the module for the specific service, Key Vault in this scenario.

### Methods

When authorizing with the Azure SDK, there are three methods to choose from:

- NewAuthorizerFromCLI
- NewAuthorizerFromEnvironment
- NewAuthorizerFromFile

#### NewAuthorizerFromCLI

If you have `az cli` installed, you can authenticate using your current az user. To show your current logged in account you can run `az account show` or `az login` to login. This may be the easiest option.

#### NewAuthorizerFromEnvironment

This will allow you to authorize using environment variables. It will look for variables belonging to different authentication mechanism in this order:

1. Client credentials
2. Client certificate
3. Username password
4. MSI

It will determine the method to use based on which of these environment variables are set:

- AZURE_SUBSCRIPTION_ID
- AZURE_TENANT_ID
- AZURE_CLIENT_ID
- AZURE_CLIENT_SECRET
- AZURE_CERTIFICATE_PATH
- AZURE_CERTIFICATE_PASSWORD
- AZURE_USERNAME
- AZURE_PASSWORD

#### NewAuthorizerFromFile

This method allows you to place credentials in a JSON file, and export an environment variable `AZURE_AUTH_LOCATION` that tells the Azure SDK where to look for the file. This file can either be created manually, or you can use the output from `az cli` when creating a new service principal. For example:

```bash
moiaune@box:~$ az ad sp create-for-rbac --sdk-auth > azureauth.json
moiaune@box:~$ cat azureauth.json
{
  "clientId": "b52dd125-9272-4b21-9862-0be667bdf6dc",
  "clientSecret": "ebc6e170-72b2-4b6f-9de2-99410964d2d0",
  "subscriptionId": "ffa52f27-be12-4cad-b1ea-c2c241b6cceb",
  "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
moiaune@box:~$ export AZURE_AUTH_LOCATION=/home/moiaune/azureauth.json
```

> NOTE: REMEMBER TO STORE THE FILE IN A SECURE LOCATION

### Example

Now, let's see this in action. I'm going to use the `NewAuthorizerFromCLI` method because its the simplest. First I need to make sure that I'm logged in to the correct account and subscription. So I run `az login` and a website will poup in my browser, telling me to log in. When thats done, you can run `az account show` to make sure that you are logged in with the correct user and subscription.

```go
package main

import (
    "github.com/Azure/azure-sdk-for-go/services/keyvault/auth"
    "github.com/Azure/azure-sdk-for-go/services/keyvault/v7.1/keyvault"
)

var (
    // for simplicity we make our client global
    client keyvault.BaseClient
)

func main() {
    // we initiate our Key Vault client
    client = keyvault.New()

    // then we initiate our authorizer
    authorizer, err := auth.NewAuthorizerFromCLI()

    // and tell our client to authenticate using that
    client.Authorizer = authorizer
}
```

This is literally it. As long as you are successfully logged into `az cli`, you are now ready to work with data in Key Vault using Go.

## 2. Fetching a Key Vault secret

Now that we have learned how to authenticate, let's try to do something usefull, like fetching a secret from Azure Key Vault.

In the Azure SDK, if you want to get a specific secret, you must provide it with a version. In most scenarios we want the latest version, so lets first write a function that will list all versions and give us the latest one. We will then write another function that fetch a secret based on latest version. We will continue to build on the code above.

```go
// ...

const (
    // name of our Key Vault
    vaultName = "example-vault-01"

    // this will build the BaseURI for our Key Vault
    vaultBaseURI = fmt.Sprintf("https://%s.%s", vaultName, azure.PublicCloud.KeyVaultDNSSuffix)
)

func getLatestVersion(secretName string) (string, error) {
    // let's fetch all versions
    list, err := client.GetSecretVersionsComplete(context.Background(), vaultBaseURI, secretName, nil)
    if err != nil {
        return "", err
    }

    var lastDate time.Time
    var lastVersion string

    // loop through all versions
    for list.NotDone() {

        v := list.Value()

        // make sure to only check for secrets that are enabled
        if *v.Attributes.Enabled {
            updated := time.Time(*v.Attributes.Updated)

            // if lastDate is not set, or current version is newer than lastDate;
            // update lastDate
            if lastDate.IsZero() || updated.After(lastDate) {
                lastDate = updated
            }

            // split the ID on '/' and get the last part which is the version hash
            parts := strings.Split(*v.ID, "/")
            lastVersion = parts[len(parts)-1]
        }

        list.Next()
    }

    return lastVersion, nil
}
```

Essentially what this code does is get all versions for a spesific secret, then loop through them to find the newest one that is also enabled. Split the ID field on the '/' and get the last part which is the version hash.

Now that we have a method to get the newest version hash, we can build our function for fetching the secret itself. We continue to build on our code from above.

```go
// ...

func getSecret(secretName string) (string, error) (
    // get latest version for our secret
    latestVersion, err := getLatestVersion(secretName)
    if err != nil {
        return "", err
    }

    // get secret itself
    secret, err := client.GetSecret(context.Background(), vaultBaseURI, secretName, latestVersion)
    if err != nil {
        return "", err
    }

    // only return the value a.k.a THE secret
    return *secret.Value, nil
}
```

First we get our latest version using our `getSecretVersion()` function, then we get the secret. The `GetSecret()` function returns a `SecretBundle` which contains some meta-data and other stuff, but in this example we're only interrested in the `Value` which is the actual secret.

If we put it all together it will look like this.

```go
package main

import (
    "context"
    "fmt"
    "strings"
    "time"

    "github.com/Azure/azure-sdk-for-go/services/keyvault/auth"
    "github.com/Azure/azure-sdk-for-go/services/keyvault/v7.1/keyvault"
    "github.com/Azure/go-autorest/autorest/azure"
)

const (
    vaultName = "example-vault-01"
    vaultBaseURI = fmt.Sprintf("https://%s.%s", vaultName, azure.PublicCloud.KeyVaultDNSSuffix)
)

var (
    client keyvault.BaseClient
)

func main() {
    client = keyvault.New()

    authorizer, err := auth.NewAuthorizerFromCLI()

    client.Authorizer = authorizer

    secretValue, err := getSecret("example-secret")
    if err != nil {
        fmt.Println("An error occured:", err)
        return
    }

    fmt.Println("Secret Value:", secretValue)
}

func getLatestVersion(secretName string) (string, error) {
    list, err := client.GetSecretVersionsComplete(context.Background(), vaultBaseURI, secretName, nil)
    if err != nil {
        return "", err
    }

    var lastDate time.Time
    var lastVersion string

    for list.NotDone() {

        v := list.Value()

        if *v.Attributes.Enabled {
            updated := time.Time(*v.Attributes.Updated)

            if lastDate.IsZero() || updated.After(lastDate) {
                lastDate = updated
            }

            parts := strings.Split(*v.ID, "/")
            lastVersion = parts[len(parts)-1]
        }

        list.Next()
    }

    return lastVersion, nil
}

func getSecret(secretName string) (string, error) (
    latestVersion, err := getLatestVersion(secretName)
    if err != nil {
        return "", err
    }

    secret, err := client.GetSecret(context.Background(), vaultBaseURI, secretName, latestVersion)
    if err != nil {
        return "", err
    }

    return *secret.Value, nil
}
```

This should output:

```bash
// Output
Secret Value: hunter2
```

## Wrapping up

So now you know how to fetch secrets from Azure Key Vault using Go. If you want to interact with other services in the Azure SDK, the process is pretty much the same.

1. Create a client from the service
2. Initiate an authorizer
3. Set the client to use the authorizer

The SDK is pretty well written and easy to understand when you just grasp the process. So it's not that difficult to dive into the source code to find answers. Tho, I still prefer proper documentation.

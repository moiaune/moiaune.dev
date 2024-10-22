---
title: "Azure Device Flow Authentication in Go"
description: "Create a simple test application in Go for authenticating to Azure with Device Flow mechanism"
tags: ['go', 'golang', 'azure']
date: 2021-10-22T11:44:49+02:00
draft: false
---

One of the usecases of Go is to create CLI tools for developers to ease their work. Often it involves creating/modifying resources or retrieving information from cloud services. In this post we're going to setup device flow authentication against AzureAD in Go.

<!--more-->

## Prerequisites

I'm assuming you already have Go installed. You also need an Azure tenant to setup authentication against. This requires that you have permissions to create "App Registrations" in Azure.

## Setting up App Registration in Azure

The first thing we must do is to create our application in Azure, which is where we set what permissions authenticated users will have and who's allowed to login through our application.

1. Go to Azure portal
2. Search for and click on "App Registrations"
3. Click "New Registrations" in the left corner
4. Give your application a name (I'm going for "Azure Device Flow Test")
5. Choose which directories are allowed to authenticate. In this example we're going to use the single tenant (first option)
6. Click "Register"

Now you have an application in Azure. By default all users in our organization has access to authenticate through our app, but the app does not have any permissions yet. For this example, we're going to give it permissions to "Azure Service Management" API, which means that users authenticated through our app can manage resources in Azure.

NOTE: Take note of "Application (client) ID" and "Directory (tenant) ID". We're going to need those later when setting up our Go CLI.

### API Permissions

For our application to be able to do anything, we must assign it some API Permissions.

1. Click on "API Permissions" in the left menu
2. Click on "Add Permission"
3. Then choose the "Azure Service Management"
4. We only have one options here, which is "user_impersonation". Check it
5. Click on "Add permissions"

Instead of having our users consent to that this application can manage Azure resources on their behalf, we're going to grant admin consent, which means that we grant access on behalf of all users of our application. NOTE: You must be an administrator in the tenant to grant admin consent.

1. Click on "Grant admin consent for <you_tenant_name>"
2. Click on "Yes" in the popup

### Authentication

Authentication in Azure uses the OAuth2 protocol, which usually takes a redirect URL that the user will be sent to when authentication was successfull. But since we're creating a CLI application, we dont have a redirect URL. Therefor, we must tell Azure that this application is a public application without a redirect URL.

1. In our App Registration, click on "Authentication" in the left menu
2. Make sure "Yes" is checked on "Allow public client flows"
3. Click "Save" at the top

As of now, our application allows anyone within our organization/directory to authenticate through this application and then give the client access to manage resources in Azure on behalf of the user. The client will only have the same access that the user actually has.

If you want to limit who can authenticate through this app even further, you can go into "Enterprise Application" and set some properties there.

1. Search for and click on "Enterprise Application"
2. Find your app in the list and click on it
3. Click on "Properties" in the left menu
4. Make sure "Assignment required" is "Yes" and click on "Save"
5. Click on "Users and groups"
6. Add users and/or groups that should be allowed access

## Setting up our Go application

So, now that we're done with the Azure part, let's setup our Go CLI.

We're going to build a CLI application that will use device flow authentication to Azure and then list all resources groups in a subscription.

### Project setup

Since this a very simple CLI application, we are only going to have a `main.go` file.

```bash
mkdir azure-device-flow-test
cd azure-device-flow-test

go mod init github.com/madsaune/azure-device-flow-test

touch main.go
```

### Device Flow Authentication

The first step is to setup authentication.

```go
// main.go

package main

import (
    "log"

    "github.com/Azure/go-autorest/autorest/azure/auth"
)

func main() {
    deviceConfig := auth.NewDeviceFlowConfig("<app_id>", "<tenant_id>")
    authorizer, err := deviceConfig.Authorizer()
    if err != nil {
        log.Fatalf("error: could not get authorizer, %v", err)
    }
}
```

This is all the code we need to authenticate to Azure. When authentication is done you can pass the `authorizer` struct to clients in the Azure SDK for Go, which we will do next.

NOTE: If you get the error "failed to get oauth token from device flow: failed to finish device auth flow: autorest/adal/devicetoken: invalid_client: AADSTS7000218: The request body must contain the following parameter: 'client_assertion' or 'client_secret'." you most likely forgot to make your App Registration in Azure public.

### List all resource groups

Now that we have our `authorizer` struct, we can pass it to our `resources.GroupsClient` when retrieving our resource groups.

```go
import (
    // ...
    "github.com/Azure/azure-sdk-for-go/services/resources/mgmt/2020-10-01/resources"
)

// ...

ctx := context.Background()

// We initialize a new GroupsClient with our subscription ID
// and the clients authorizer to the one we got from the
// device flow authentication, so that the GroupsClient can
// authenticate as us
c := resources.NewGroupsClient("<subscription_id>")
c.Authorizer = authorizer

// ListComplete will give us all resource groups within the
// subscription that the GroupsClient is initialized for
groupList, err := c.ListComplete(ctx, "", nil)
if err != nil {
    log.Fatalf("error: could not list resource groups, %v", err)
}

// Loop through the result
for groupList.NotDone() {
    // Retrieve the current Group struct from the iterator
    group := groupList.Value()

    // Let's print the name and location of the resource group
    fmt.Printf("- %s (Location: %s)\n", *group.Name, *group.Location)

    // NextWithContext() will return an error if there are noe more results.
    // We then want to exist
    if err := groupList.NextWithContext(ctx); err != nil {
        break
    }
}
```

You must provide your own `app_id`, `tenant_id` and `subscription_id`. To see the whole code, you can find it on Github [here](https://github.com/madsaune/azure-device-flow-test).

### Build and run

If we try to build and run our application it will behave something like this.

```bash
mm@box:~/code/azure-device-flow-test$ go build
mm@box:~/code/azure-device-flow-test$ ./azure-device-flow-test
mm@box:~/code/azure-device-flow-test$ 2021/10/22 12:26:42 To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code CLEUYSFZK to authenticate.
- my-example-rg-1 (Location: westeurope)
- my-example-rg-2 (Location: westeurope)
- my-example-rg-3 (Location: westeurope)
- my-example-rg-4 (Location: westeurope)
```

On line 3 you must open the URL in a browser and enter the code in the box, then authenticate as a user in your tenant. When that's done, it will list resource groups in the specified subscription.

## Next steps

Now we have a great foundation to start working with resources in Azure through Go. We could initialize more clients and use the same authorizer.

One thing that is a must to implement is getting `app_id`, `tenant_id` and `subscription_id` from environment variables. These ID's should not be hardcoded in your application.

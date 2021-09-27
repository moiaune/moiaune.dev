---
title: 'Get status code for failed webrequests in Powershell'
date: 2021-09-23T21:35:29+02:00
draft: false
---

If you are sending web requests with Powershell you will notice that if your requests fails, that is if it returns any status code other than 2xx, it will thrown an error. Now, how do get the details of the failed request?

In Powershell, when you use `Invoke-WebRequest` or `Invoke-RestMethod`, it will give you details about the failed request in the `$_.Exception.Response` object. Let's say you want to know if the error is because of a bad request or an internal server error, you can do this:

```powershell
try {
    $response = Invoke-RestMethod -Uri "https://jsonplaceholder.typicode.com/users/11"
} catch {
    $StatusCode = $_.Exception.Response.StatusCode

    if ($StatusCode -eq [System.Net.HttpStatusCode]::NotFound) {
        Write-Error "User was not found!"
    } elseif ($StatusCode -eq [System.Net.HttpStatusCode]::InternalServerError) {
        Write-Error "InternalServerError: Something went wrong on the backend!"
    } else {
        Write-Error "Expected 200, got $([int]$StatusCode)"
    }
}
```

If you want to compare the status code as an integer you can just cast the status code, which is as you can see from the example above, a System.Net.HttpStatusCode enum.

```powershell
try {
    $response = Invoke-RestMethod -Uri "https://jsonplaceholder.typicode.com/users/11"
} catch {
    $StatusCode = [int]$_.Exception.Response.StatusCode

    if ($StatusCode -eq 404) {
        Write-Error "User was not found!"
    } elseif ($StatusCode -eq 500) {
        Write-Error "InternalServerError: Something went wrong on the backend!"
    } else {
        Write-Error "Expected 200, got $([int]$StatusCode)"
    }
}
```

As always when using namespaces in Powershell you can define them at top and it will save you some typing.

```powershell
using namespace System.Net

try {
    $response = Invoke-RestMethod -Uri "https://jsonplaceholder.typicode.com/users/11"
} catch {
    $StatusCode = $_.Exception.Response.StatusCode

    if ($StatusCode -eq [HttpStatusCode]::NotFound) {
        Write-Error "User was not found!"
    } elseif ($StatusCode -eq [HttpStatusCode]::InternalServerError) {
        Write-Error "InternalServerError: Something went wrong on the backend!"
    } else {
        Write-Error "Expected 200, got $([int]$StatusCode)"
    }
}
```

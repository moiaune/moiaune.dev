---
title: "Golang: Azure Functions Blob Storage Output Binding"
date: 2024-02-26T22:19:04+01:00
draft: false
---

Lately, I've been setting up an Azure Function App with a custom handler written in Go. One of my functions needs to download a file from an external URL and then upload that file to Azure Blob Storage. Unfortunately, neither the documentation on Microsoft Learn or the examples on Github mentions how to use Blob Storage as output binding for custom handlers. So I decided to do a little write up on how I solved it.


There are two types of files that you can upload:

- binary
- textfile

If you are uploading a binary the Azure Function App host expects the file as byte array (`[]byte`) otherwise it expects the file as base64 encoded string.

> NOTE: I dont handle errors in these examples to keep the code short, but you should always handle errors!

## As binary

To upload a file a binary to Blob Storage using output bindings we need to specify in the `function.json` that the `dataType` will be 'binary'. Then when we return our custom handler payload to Azure Function App, the `returnValue` must be a byte array (`[]byte`).

```json
// file: function.json

{
    "bindings": [
        // ...
        {
            "name": "$return",
            "type": "blob",
            "direction": "out",
            "path": "reports/my_report.csv",
            "connection": "AzureWebJobsStorage",
            "dataType": "binary"
        }
    ]
}
```

```go
// file: handler.go

type BlobOutputBinding struct {
    ReturnValue interface{}
}

func DownloadHandler(w http.ResponseWriter, r *http.Request) {

    // ... Logic for downloading file

    // 1. Convert our response body (a.k.a downloaded file) to []byte
    data, _ := io.ReadAll(resp.Body)

    // 2. Create our custom handler response payload
    // since we named our blob output `$return`, we can use the `returnValue` instead of `Outputs["outblob"]`
    binding := BlobOutputBinding{
        ReturnValue: data
    }

    // 3. convert the binding to JSON
	response, _ := json.Marshal(binding)

    // 4. Respond to Azure Function with out binding
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	w.Write(response)
}
```

## As text file

If we want to upload a text file the `dataType` in `function.json` must be 'string', or we can leave it out because 'string' is the default value. When `dataType` is 'string' the Azure Function App expects our custom handler to return the file as a base64 encoded string.

```json
// file: function.json

{
    "bindings": [
        // ...
        {
            "name": "$return",
            "type": "blob",
            "direction": "out",
            "path": "reports/my_report.csv",
            "connection": "AzureWebJobsStorage",
        }
    ]
}
```

```go
// file: handler.go

type BlobOutputBinding struct {
    ReturnValue interface{}
}

func DownloadHandler(w http.ResponseWriter, r *http.Request) {

    // ... logic for downloading file

    // 1. convert our response body (a.k.a downloaded file) to []byte
    data, _ := io.ReadAll(resp.Body)

    // 2. convert to base64
	encoded := base64.StdEncoding.EncodeToString(data)

    // 3. create our custom handler response payload
    // since we named our blob output `$return`, we can use the `returnValue` instead of `Outputs["outblob"]`
    binding := BlobOutputBinding{
        ReturnValue: encoded
    }

    // 3. convert the binding to JSON
	response, _ := json.Marshal(binding)

    // 4. respond to Azure Function with out binding
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	w.Write(response)
}
```



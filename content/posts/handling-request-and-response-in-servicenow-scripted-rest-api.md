---
title: "Handling Request and Response in Servicenow Scripted REST API"
description: "How to handle request and response in ServiceNow Scripted REST API"
date: 2021-10-28T16:00:51+02:00
tags: ['servicenow', 'javascript', 'api', 'scripted-rest-api']
draft: true
---

## Request

```javascript
(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {

    // Get the request body
    var requestBody = request.body.data;

    // Get the request headers
    var requestHeaders = request.headers;

    // Get specific header
    var acceptHeader = request.getHeader('accept');

    // Get query parameters
    var requestQueryParams = request.queryParams;

    // Get path parameters
    var requestPathParams = request.pathParams;

})(request, response);
```

### Validating JSON properties

Since we often cannot control what the client sends us, validating the JSON request body is very handy. This way we can tell the client that the request they sent us is not what we were expecting and its good first step to eliminate bugs.

```javascript
// Simple function to validate data against a set of rules in a flat JSON object
function validateRequest(data, rules) {
    var dataKeys = Object.keys(data).sort();
    var rulesKeys = Object.keys(rules).sort();

    if (dataKeys.toString() !== rulesKeys.toString()) {
        return false;
    }

    for(var i = 0; i < rulesKeys.length; i += 1) {
        var key = rulesKeys[i];

        if (typeof data[key] === "undefined" || typeof data[key] !== rules[key]) {
            return false;
        }
    }

    return true;
}

var rules = {
    name: "string",
    value: "string",
    active: "boolean"
};

if (!validateRequest(request.body.data, rules)) {
    var err = new sn_ws_err.SericeError();
    err.setStatus(400);
    err.setMessage('Invalid request body');
    err.setDetail('Request body is not in expected format');

    reponse.setError(err);
    return;
}
```

## Response

When creating a REST API it's important how you handle your responses, because thats the only way your client will know what has happend.

When everything goes well, we want to sent a response status in the 2XX range. Most commonly is 200 and 201. You also want to let your clients know which format the response is in. Most common these days are `application/json`.

```javascript
// 201 Created
response.setContentType('application/json');
response.setStatus(201);
response.setBody({
    message: "Your object was created",
    object: {
        name: "My object",
        value: "some value",
    }
});

// 200 OK
response.setContentType('application/json');
response.setStatus(200);
response.setBody({
    name: "My object",
    value: "some value",
});
```

There is alot that can go wrong when handling data that is sent by clients, and its a good practice to give good error responses so the clients known what went wrong. ServiceNow has some built in error types show below.

```javascript
// BadRequest error - 400
response.setError(new sn_ws_err.BadRequestError('Request was malformatted'));

// NotFound error - 404
response.setError(new sn_ws_err.NotFoundError('Object was not found'));

// NotAcceptable error - 406
response.setError(new sn_ws_err.NotAcceptableError('Accept header is not a supported type'));

// ConflictError - 409
response.setError(new sn_ws_err.ConflictError('There was a conflict'));

// UnsupportedMediaType error - 415
response.setError(new sn_ws_err.UnsupportedMediaTypeError('The requested media type is not supported'));
```

You can also create custom errors.

```javascript
// Custom error
var err = new sn_ws_err.ServiceError();
err.setStatus(500);
err.setMessage('Internal Server Error');
err.setDetail('Something went wrong with the request');

response.setError(err);
```
By using the response object's error handler, you make sure that error are reported consitently. It will follow the standard ServiceNow error format which looks like this.

```json
{
    "error": {
        "detail": "",
        "message": ""
    },
    "status": "failure"
}
```

## Example

```javascript
(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
    // Check if correct content type

    // Read request body

    // Do something

    // Handle errors

    // Respond
})(request, response);
```

## Resources

[https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/sn_ws-namespace/c_RESTAPIRequest](https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/sn_ws-namespace/c_RESTAPIRequest)
[https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/sn_ws-namespace/c_RESTAPIRequestBody](https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/sn_ws-namespace/c_RESTAPIRequestBody)
[https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/sn_ws-namespace/c_RESTAPIResponse](https://developer.servicenow.com/dev.do#!/reference/api/quebec/server/sn_ws-namespace/c_RESTAPIResponse)
[https://developer.servicenow.com/dev.do#!/learn/courses/quebec/app_store_learnv2_rest_quebec_rest_integrations/app_store_learnv2_rest_quebec_scripted_rest_apis/app_store_learnv2_rest_quebec_scripted_rest_api_error_objects](https://developer.servicenow.com/dev.do#!/learn/courses/quebec/app_store_learnv2_rest_quebec_rest_integrations/app_store_learnv2_rest_quebec_scripted_rest_apis/app_store_learnv2_rest_quebec_scripted_rest_api_error_objects)

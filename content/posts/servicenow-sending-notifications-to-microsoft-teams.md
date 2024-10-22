---
title: "ServiceNow: Sending notifications to Microsoft Teams"
description: "Microsoft Teams support different Connections and one of the simpliest is 'Incomming Webhook' which gives you a URL that you can POST to with a correctly configured JSON body and the result will be displayed in your specified channel. In this guide we’ll set this up and POST to it from Service-Now when an incident is created or updated."
tags: ["servicenow", "javascript"]
date: 2018-11-29T00:00:00+01:00
draft: false
---

Microsoft Teams support different Connections and one of the simpliest is "Incomming Webhook" which gives you a URL that you can POST to with a correctly configured JSON body and the result will be displayed in your specified channel. In this guide we’ll set this up and POST to it from Service-Now when an incident is created or updated.

<!--more-->

## Setting up Connections in Microsoft Teams

To setup an "Incomming Webhook" for a specific channel do the follow:

1. Place your cursor over the channel name and click the three dotts to the right.
2. Choose "Connections".
3. Find "Incomming Webhooks" in the list of connectors and click "Configure".
4. Give the connection a name and picture (optional) and click "Create".

Take note of the URL in the greyed out box starting with "https://outlook.office.com/webhook/..". We will use this in ServiceNow later.

## Creating an "Outbound REST Message" in ServiceNow

To be able to send an outbound REST message in Service-Now you must first create and configure one. In your application sidebar go to **System Web Services → Outbound → REST Message** and create a new REST Message. Fill out the required information.

**Name** – We will refeer to this in our Business Rule later
**Endpoint** – Paste in the URL from your "Incomming Webhook" in Microsoft Teams

By default Service-Now will only create a GET method for our REST Message. Edit your newly created REST Message and scroll down to "HTTP Methods" and create a new one for POST. Fill out the required information.

**Name** – Just set the name to be ‘POST’ (without quotes)
**HTTP Method** – This must be set to POST, which is the method Microsoft Teams expects us to use
**Endpoint** – Paste in the same URL as before

Now our Outbound REST Message is ready and we can create a Business Rule to trigger it.

## Trigger messages with Business Rules in ServiceNow

Now that we have the basics setup, we can create a Business Rule in Service-Now to trigger messages based on our criteria. In this guide we will setup a business rule to trigger when an incident is priority 1 and assigned to the ‘Monitoring’ group. The webhook we setup earlier goes to an ‘Alert’ channel in their team. Browse to **System Definition → Business Rules → New** to create a new business rule.

First we must name our business rule. If you are going to have a lot of Teams and channels I would recommend to use a naming scheme I.e `Teams <team>_<channel> <tag>`. Just remember that the maximum length is 40 characters. So for our business rule it will be "Teams monit_alert pri1".

Then we must define which table to run the business rule against. Since we are targeting incidents, we choose "Incident [incident]".

### When to run

Next, we’re going to setup our filter for when to run under the tab “When to run”. For this demo its going to be a fairly simple filter, but you could make all sorts of advanced filters for when to trigger. Lets have a look at what the different properties does.

#### When

All business rules run on database operations (insert, update, delete or query). Here we can set when we want the business rule to trigger. Before, after or async with database operation. I our demo we will use ‘after’ just so we are sure the data is correctly written in the database before we are sending it outside Service-Now. If we used ‘before’ there might be some other business rule that manipulate the data after our script but before it is written to the database and so the data we have access to might not be the correct one after all.

#### Order

If you have multiple business rule with the same filtering, you can decide in which order you want them to execute. Lowest executes first and highest last. We will keep our default (100).

#### Filter Conditions

This is were most of the power lies. Here we can set our colum ⇒ value filters. In our demo we want to set these conditions:

**Active** is true
**Assignment group** is ‘Monitoring’
**Assigned to** is empty
**Priority** is ‘1 – Critical’

And we want it to execute on both database **inserts** and **updates**. The reason we specify Assigned to to be empty is if we did not the rule would execute everytime the incident is updated, even though it is already assigned.

### Time for some code

At the top of your form, to the right, you will se a checkbox for Advanced. Make sure it’s checked to see the **Advanced** tab. This is where the magic happens. Inside the **Advanced** tab you have a script field where we can enter some code to execute. Here’s the code and I will break it down for you.

```javascript
(function executeRule(current, previous) {

  if (current == null) {
    gs.log("OUTBOUND REST - Teams monit_alert pri1 - Current is not defined");
    return;
  }

  var requestBody;
  var responseBody;
  var status;
  var r;
  var desc = current.description.toString().replace(/(?:\r\n|\r|\n)/g, ' ');
  var shortened_desc = (desc.length > 140) ? desc.substring(0, 140) + "[...]" : desc;
  var link_appl = encodeURI("https://<instance>.service-now.com/nav_to.do?uri=%2F" + current.sys_class_name + ".do%3Fsys_id%3D" + current.sys_id);

  var body = {
    "@type": "MessageCard",
    "@context": "http://schema.org/extensions",
    "themeColor": "0076D7",
    "summary": "New incident has been opened",
    "sections": [{
      "activityTitle": "New Incident has been opened",
      "activitySubtitle": "Monitoring",
      "activityImage": "https://<instance>.service-now.com/<your_logo>.png",
      "facts": [{
        "name": "Case ID",
        "value": current.number.toString()
      },
      {
        "name": "Title",
        "value": current.short_description.toString()
      },
      {
        "name": "Company",
        "value": current.company.name.toString()
      },
      {
        "name": "Description",
        "value": shortened_desc
      }],
      "markdown": true
    }],
      "potentialAction": [{
        "@type": "OpenUri",
        "name": "View in Service-Now",
        "targets": [
          {
            "os": "default",
            "uri": link_appl
          }
        ]
    }]
  };

  try {
    r = new sn_ws.RESTMessageV2("Teams monit_alert pri1", "post");
    r.setRequestBody(JSON.stringify(body));

    resp = r.execute();
    responseBody = resp.haveError() ? resp.getErrorMessage() : resp.getBody();
    status = resp.getStatusCode();
  } catch(ex) {
    responseBody = ex.getMessage();
    status = '500';
  } finally {
    requestBody = r ? r.getRequestBody() : null;
  }

  gs.log("OUTBOUND REST - Teams monit_alert pri1 - Request Body: " + requestBody);
  gs.log("OUTBOUND REST - Teams monit_alert pri1 - Response: " + responseBody);
  gs.log("OUTBOUND REST - Teams monit_alert pri1 - HTTP Status: " + status);

})(current, previous);
```

**Line 3-6:** This just makes sure that we actually have a `current` object. This is where the incident details are stored and we can make changes to. If it’s not set we’re aborting the whole thing.

**Line 12-13:** Here we are removing any newlines from the description and only includes the first 140 characters. Note that these changes are not written to database, just in the REST message.

**Line 14:** This will generate a URL so that we can link to the incident in Service-Now. Replace `<instance>` with your instance name.

**Line 16-53:** This is a JSON object that defines the look and content of our Microsoft Teams message. This is Microsoft Teams specifics and you can read more about it [here](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/cards/cards) and see some examples and test your JSON object [here](https://messagecardplayground.azurewebsites.net/). Remember to replace `<instance>` with your instance name and `<your_logo>` to your logo path in the _ActivityImage_ property.

One important thing to notice is that we need to append `.toString()` when referring to attributes in the `current` object because they return objects and will not render our JSON correctly when we conver it to text later. You could also assign the `current` attributes to a variable and then refeer to the variable in the JSON, but thats just unnecessary code in my opinion.

**Line 55-67:** This is where we actually send the message to Microsoft Teams and we have encapsualted it in a try/catch statement. On line 56 we instantiate a new REST message object and refeer to the REST Message we created earlier. Use the name as the first parameter and then which HTTP method to use.

Next we convert our JSON to a string and pass it to the request body on line 57. Then we execute the request on line 59 and then store our response on line 60 and 61. Response code ‘200’ indicates that the request was successful.

**Line 69-71:** Here we just print our script log so we can debug if its not working. Your script log is located here **System Logs → System Log → Script Log Statement**.

### Finalize it

Once our filter is set and we have inserted our script we just need to save or update the business rule. Make sure that **Active** is checked.

## Testing

Now that we have everything setup we can start testing it. Lets create a new incident with a caller, priority 1, assigned to group ‘Monitoring’, no assignee and a dummy short description and description. If everything was correctly setup you should immediatly see the card in your channel of choice in Microsoft Teams with the information you provided when creating the incident.

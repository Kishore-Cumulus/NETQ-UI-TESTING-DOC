---
description: >-
  Using cypress mocks to develop or test features for which the BE API
  integration is not completed or the premise is not present.
---

# Developing/Testing features

There are times where we need to develop features even before the Backend part is done. In those cases we generally mock the data at various layers of the app like the api-service layer, selector layer  or even inside the component itself. All the above methods work with no problem but let's say you want to test the  feature for a error response, we will be commenting the old mocks and replace then with new mocks, but managing these mocks at different places and all the different mocks will become difficult.

For the above problem, we can use cypress as a way to mock the responses and also have our code tested for different cases in a more manageable and scalable way. 

Let's go over a use case where we can use the above approach to set the mock data.

Problem statement:  
Add a new feature to show disk space availability at the upload image modal. As the BE implementation is not done the API contract can be decided upfront between the two parties. Let's say we decided on the contract for this particular case

**method**:       GET  
**url**:               netq/lcm/v1/image/disk\_usage  
**request**:      NA  
**response**: 

```text
{
      "total_disk_space": 8765,
      "available_disk_space": 7000
}
```

Once the API contract is done we can change this to cypress mock schema. On a high note, cypress auto record mock should be in the following schema.

{% tabs %}
{% tab title="POST mock" %}
```javascript
  {
    "url": "**/netq/telemetry/v1/object/snapshot/compare",
    "method": "POST",
    "status": 200,
    "headers": {},
    "body": {
      "before_snapshot_name": "AFTER_ssee--h",
      "after_snapshot_name": "AFTER_MUTE-spine3-1",
      "brief": true
    },
    "response": {
      "after_snapshot_name": "AFTER_MUTE-spine3-1",
      "before_snapshot_name": "AFTER_ssee--h",
      "status": "Success"
    }
  },
```
{% endtab %}

{% tab title="GET mock" %}
```javascript
  {
    "url": "**/netq/user/v1/profile",
    "method": "GET",
    "status": 200,
    "headers": {},
    "body": null,
    "response": {
      "date_created": "03/30/2020",
      "default_workbench": {
        "workbench_id": "DEFAULT",
        "workspace_id": "DEFAULT"
      },
      "email": "admin",
      "enable_alarm_notifications": true,
      "first_name": "Admin",
      "id": "admin",
      "is_active": true,
      "is_ldap_user": false,
      "last_login": "05/12/2020",
      "last_name": "",
      "preferences": {
        "date_format": "M/d/yy h:mm a",
        "theme": "jade",
        "timezone": "+0530"
      },
      "role": "admin",
      "terms_of_use_accepted": true
    }
  },
```
{% endtab %}
{% endtabs %}

the schema is mostly self explanatory, and for GET case you can have body field to be null.

With the above information, let's construct our mock for disk space API and it would be something like below

```javascript
  {
    "url": "**/netq/lcm/v1/image/disk_usage",
    "method": "GET",
    "status": 200,
    "headers": {},
    "body": null,
    "response": {
      "total_disk_space": 8765,
      "available_disk_space": 7000
    }
  },
```

Now to add this mock, we can have two options

1. `@default.json`
2.  `spec_name.spec.json` 
3.  mock in the test case itself

Both options are totally valid. One suggested way is to add default response mock at the `default.json` file and add specific case mocks i.e 400 error code responses, empty responses in the `spec_name`.`spec.json`

{% hint style="info" %}
mocks in default json will be by default available to all specs
{% endhint %}

Now we can develop our feature by just adding a normal test case in whichever spec file you like, the test case is not required full blown. we can just add cy.visit\('routename'\) and manually click/navigate inside the cypress browser. As a next step you can include these steps in your test file for automated testing.

And say you want to test a case for avaliable\_disk\_space is 0. This can be done easily by just writing a test case let's say `validate case for zero available_disk_space`

  for this you can add the needed response in the `spec_name.spec.json`

{% code title="mocks/spec\_name.spec.json" %}
```javascript
{
    "validate case for zero available_disk_space": [
      {
        "url": "**/netq/lcm/v1/image/disk_usage",
        "method": "GET",
        "status": 200,
        "headers": {},
        "body": null,
        "response": {
        "total_disk_space": 8765,
        "available_disk_space": 0
      }
    }
  ]
}
```
{% endcode %}

Now the cases are more declarative and manageable. 

and a third option is to add the mock data in your test case itself if it needs some more configuration like waiting until the request is done or any other case.  


```javascript
  const responses = [
    {
      "url": "**/netq/lcm/v1/credential",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": []
    }
  ];
  responses.forEach(response => {
    cy.route({
      method: response.method,
      url: response.url,
      response: response.response,
      status: response.status
    });
  });
```

Take this a starting point and feel free to tweak this to your requirements and share them to the team


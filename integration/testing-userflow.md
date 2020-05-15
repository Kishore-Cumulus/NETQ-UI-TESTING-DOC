# Testing Userflow

## Jump start

{% code title="cypress/integration/\*module\*/\*spec\_name\*.spec.ts" %}
```typescript
import autoRecord from '../../plugins/cypress-autorecord';

describe('*spec name*', () => {
  autoRecord(__filename);

  before(() => {
    cy.login();
    cy.saveLocalStorage();
  });

  beforeEach(() => {
    cy.restoreLocalStorage();
  });
  
  it('*test name*', () => {
  }
}

```
{% endcode %}



## Implementation\(step by step\)

* Go to `cypress/integration` add a new folder based on the module testing let's say management, profile, lcm.
* create a spec file with name `*spec_name*.spec.ts` with the below content.

{% tabs %}
{% tab title="structure" %}
{% code title="cypress/integration/\*module\*/\*spec\_name\*.spec.ts" %}
```typescript
import autoRecord from '../../plugins/cypress-autorecord';

describe('*spec name*', () => {
  autoRecord(__filename);

  before(() => {
    cy.login();
    cy.saveLocalStorage();
  });

  beforeEach(() => {
    cy.restoreLocalStorage();
  });
}

```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="cypress/integration/lcmhome/switch\_upgrade\_modal\_navigation.spec.ts" %}
```typescript
import autoRecord from '../../plugins/cypress-autorecord';

describe('switch upgrade modal navigation', () => {
  autoRecord(__filename);

  before(() => {
    cy.login();
    cy.saveLocalStorage();
  });

  beforeEach(() => {
    cy.restoreLocalStorage();
  });
});

```
{% endcode %}
{% endtab %}
{% endtabs %}

* create a new test case with 

{% tabs %}
{% tab title="structure" %}
```typescript
it('*test name*', () => {
}
```
{% endtab %}

{% tab title="example" %}
```typescript
it('should open switch upgrade modal', () => {
    cy.visit('/lcmhome');
    cy.get('[routerlink="lcmhome/switches"]').click();
    cy.get('hzn-item-list [type="checkbox"]').should('have.length.greaterThan', 0);
    cy.get('hzn-item-list [type="checkbox"]').first().click();
    cy.get('[data-title="Upgrade Switches"]').should('exist').click();
    cy.get('hzn-upgrade-switches').should('exist');
}
```
{% endtab %}
{% endtabs %}

#### Auto record checks for the spec\_name.spec.json inside the mocks folder and if it doesn't find it will directly call the server and seed the data it got from the response and the next time onward it will fetch from the file. it's similar to cache strategy. We can go into the file and change the data and new keys with  different data and error cases.

#### If you want to have a mock server from the start, use the following approach where you create the mock file and seed the data yourselves

* create a new file inside `cypress/mocks` folder with the name `*spec_name.spec.json*` . Please ensure both the new files inside integration and mocks have the same name. In case you changed the file name in integration, make sure to change the same in the mocks folder.
* create a new object with a key  as `*test name*` and pass an array of all the APIs to be mocked

{% tabs %}
{% tab title="structure" %}
{% code title="cypress/mocks/\*spec\_name\*.spec.json" %}
```text
{
    "*test name*": [
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/premises/v1",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": {
        "is_cloud": true,
        "premises": [
          {
            "id": 0,
            "is_active": false,
            "name": "OPID0",
            "namespace": "NAN",
            "tag": "TAG0",
            "timezone": "IST"
          }
        ],
        "user_customer": {
          "id": 0,
          "is_ldap_enabled": false,
          "name": "Cumulus Internal"
        }
      }
    },
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/premises/v1",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": {
        "is_cloud": true,
        "premises": [
          {
            "id": 0,
            "is_active": false,
            "name": "OPID0",
            "namespace": "NAN",
            "tag": "TAG0",
            "timezone": "IST"
          }
        ],
        "user_customer": {
          "id": 0,
          "is_ldap_enabled": false,
          "name": "Cumulus Internal"
        }
      }
    },
    ]
}
```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="cypress/mocks/switch\_upgrade\_modal\_navigation.spec.json" %}
```typescript
{
  "should open modal from home page": [
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/premises/v1",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": {
        "is_cloud": true,
        "premises": [
          {
            "id": 0,
            "is_active": false,
            "name": "OPID0",
            "namespace": "NAN",
            "tag": "TAG0",
            "timezone": "IST"
          }
        ],
        "user_customer": {
          "id": 0,
          "is_ldap_enabled": false,
          "name": "Cumulus Internal"
        }
      }
    },
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/user/v1/workbench?show_public=true",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": [
        {
          "creator": "admin",
          "data": {
            "name": "Demo"
          },
          "date_created": 1588025720,
          "is_public": false,
          "last_edited": 1588025720,
          "owner": "admin",
          "workbench_id": "10735961013874138",
          "workbench_name": "Demo",
          "workspace_id": "DEFAULT"
        },
        {
          "creator": "",
          "data": {},
          "date_created": 1585588147,
          "is_public": true,
          "last_edited": 1585588147,
          "owner": "",
          "workbench_id": "DEFAULT",
          "workbench_name": "Cumulus Workbench",
          "workspace_id": "DEFAULT"
        }
      ]
    },
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/user/v1/profile",
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
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/external-premises/v1/",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": []
    },
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/user/v1/workbench/workspace/DEFAULT/workbench/DEFAULT",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": {
        "cards": [
          {
            "data": {
              "editable": false,
              "id": "WYS5MTV"
            },
            "name": "[CARD] GENERIC HARDWARE",
            "size": "card"
          },
          {
            "data": {
              "editable": false,
              "id": "VAC6SNH"
            },
            "name": "[CARD] INVENTORY",
            "size": "card"
          },
          {
            "data": {
              "editable": false,
              "id": "VAC6SNC"
            },
            "name": "[CARD] ALERTS",
            "size": "card"
          },
          {
            "data": {
              "editable": false,
              "id": "WYS5MTP"
            },
            "name": "[CARD] INFO",
            "size": "card"
          },
          {
            "data": {
              "editable": false,
              "id": "VAC6SNS"
            },
            "name": "[CARD] NETWORK HEALTH",
            "size": "card"
          }
        ],
        "data": {
          "name": "Cumulus Workbench"
        },
        "is_public": true,
        "owner": "",
        "workbench_id": "DEFAULT",
        "workbench_name": "Cumulus Workbench",
        "workspace_id": "DEFAULT"
      }
    },
    {
      "url": "https://api.gui1.netqdev.cumulusnetworks.com/netq/telemetry/v1/object/check/nw_health?bucket=1&details=l1&duration=24&time=1589312258",
      "method": "GET",
      "status": 200,
      "headers": {},
      "body": null,
      "response": {
        "Summary": {
          "move_per": 0,
          "summary": "low",
          "trend": "Low"
        },
        "data": {
          "Overall": [
            {
              "date": 1589254658,
              "value": 83
            }
          ]
        }
      }
    }
  ],
}
```
{% endcode %}
{% endtab %}
{% endtabs %}


# Testing Selectors

## Jump start

Use the below code and replace \*\* with your module names. For step by step approach refer [this](testing-reducers.md#integration-step-by-step)

```typescript
import * as selectors from '*./testing.selector*';
import { initialState } from '*./testing.reducer*';

describe('*TestingSelector*', () => {
  let state;
  beforeEach(() => {
    state = initialState;
  });

  it('should selectValue', () => {
    const selectValue = selectors.selectValue('randomXYZ');
    expect(selectValue.projector(state)).toEqual(null);
  });
  
  it('should *selector1*', () => {
    const *selector1* = selectors.*selector1*;
    const *someProperty* = {
    };
    state = {
      ...state,
      *someProperty*,
    };
    expect(*selector1*.projector(state)).toEqual(*someProperty*);
  });
});
```

## Concepts

Selector testing is similar to reducer since selectors are also simple functions. Along with that we will be using projector method on the selector to mimic the values of the intermediate selectors

```typescript
export const mySelector = () => createSelector(
  organisationSelector,
  teamSelector,
  (org, team) => (org + ' ' + team)
);
```

if we take the above example to test the `mySelector()` we can use projector method like `mySelector().projector('Cumulus', 'NetQ UI').expect('Cumulus NetQ')`   

##  Implementation\(step by step\)

* create a new file named \*testing.selector.spec.ts\* and copy paste the below and replace it with your relevant things wherever it starts with \* and ends with \*

{% tabs %}
{% tab title="structure" %}
{% code title="store/\*testing.selector.spec.ts\*" %}
```typescript
import * as selectors from '*./testing.selector*';
import { initialState } from '*./testing.reducer*';

describe('*TestingSelector*', () => {
  let state;
  beforeEach(() => {
    state = initialState;
  });

  it('should selectValue', () => {
    const selectValue = selectors.selectValue('randomXYZ');
    expect(selectValue.projector(state)).toEqual(null);
  });
});

```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="store/external-premises/external-premises.selector.spec.ts" %}
```typescript
import { ExternalPremisesStates } from '../../models';
import * as selectors from './external-premises.selector';

describe('ExternalPremisesSelector', () => {
  let state;
  beforeEach(() => {
    state = {
      addedExternalPremises: null,
      allAvailableExternalPremises: null,
      toBeAddedExternalPremises: null,
    };
  });

  it('should selectValue', () => {
    const selectValue = selectors.selectValue('addedExternalPremises');
    expect(selectValue.projector(state)).toEqual(null);
  });

});

```
{% endcode %}
{% endtab %}
{% endtabs %}

* writing a unit test. There are  three/four steps in a unit test for selector

  * selecting the selector that to be tested
  * expected value from the selector.
  * de-structure state along with that value
  * passing the state into projector of the selector and asserting against expected value.

{% tabs %}
{% tab title="structure" %}
{% code title="store/\*testing.selector.spec.ts\*" %}
```typescript
it('should *selector1*', () => {
  const *selector1* = selectors.*selector1*;
  const *someProperty* = {
  };
  state = {
    ...state,
    *someProperty*,
  };
  expect(*selector1*.projector(state)).toEqual(*someProperty*);
});
```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="store/external-premises/external-premises.selector.spec.ts" %}
```typescript
  it('should selectAllAvailableExternalPremises', () => {
    const selectAllAvailableExternalPremises = selectors.selectAllAvailableExternalPremises;
    const allAvailableExternalPremises = {
      state: ExternalPremisesStates.DEFAULT,
      error: null,
      data: [ {
        id: 1
      }],
    };
    state = {
      ...state,
      allAvailableExternalPremises,
    };
    expect(selectAllAvailableExternalPremises.projector(state)).toEqual(allAvailableExternalPremises);
  });
```
{% endcode %}
{% endtab %}
{% endtabs %}


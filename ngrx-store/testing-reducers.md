# Testing Reducers

## Jump start

Use the below code and replace \*\* with your module names. For step by step approach refer [this](testing-reducers.md#implementation-step-by-step)

```typescript
import * as actions from '*./testing.actions*';
import reducer, { initialState } from '*./testing.reducer*';
import { MockData } from './mocks/mock-data';

describe('*TestingReducer*', () => {
  let state;
  beforeEach(() => {
    state = initialState
  });

  it('should return initial state', () => {

    const action = {type: ''};
    const expected = state;
    expect(reducer(undefined, action)).toEqual(expected);
  });
  
  it('test name', () => {
    const data = MockData.*ACTION_NAME_SUCCESS_INDEX*;
    const action = new actions.*Action1Success*(data);
    state = {
      ...state
    };
    const expected = {
      ...state
    };
    expect(reducer(state, action)).toEqual(expected);
  });
});


```

## Concepts

Testing reducers is mostly straight forward. I didn't find any reference in ngrx  regarding this but if we treat reducers as pure functions\(which they are\), testing will be similar to testing normal methods

## Implementation\(step by step\)

* create a new file named \*testing.reducer.spec.ts\* and copy paste the below and replace it with your relevant things wherever it starts with \* and ends with \*

{% tabs %}
{% tab title="structure" %}
{% code title="store/\*testing.reducer.spec.ts\*" %}
```typescript
import * as actions from '*./testing.actions*';
import reducer, { initialState } from '*./testing.reducer*';
import { MockData } from './mocks/mock-data';

describe('*TestingReducer*', () => {
  let state;
  beforeEach(() => {
    state = initialState
  });

  it('should return initial state', () => {

    const action = {type: ''};
    const expected = state;
    expect(reducer(undefined, action)).toEqual(expected);
  });
});

```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="store/external-premises/external-premises.reducer.spec.ts" %}
```typescript
import { ExternalPremisesStates } from '../../models';

import * as actions from './external-premises.actions';
import reducer from './external-premises.reducer';
import { MockData } from './tests/mock-data';

describe('ExternalPremisesReducer', () => {
  let state;
  beforeEach(() => {
    state = {
      addedExternalPremises: null,
      allAvailableExternalPremises: null,
      toBeAddedExternalPremises: null,
    };
  });

  it('should return initial state', () => {

    const action = {type: ''};
    const expected = state;
    expect(reducer(undefined, action)).toEqual(expected);
  });
});

```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Replace \*...\* with your specific module names at all places
{% endhint %}

* writing a unit test. Commonly there are  three/four steps in a unit test for reducer
  * loading the data for the action from mock
  * passing the data to action object and instantiating it.
  * setting the current state
  * setting the expected state

{% tabs %}
{% tab title="structure" %}
{% code title="store/\*testing.reducer.spec.ts\*" %}
```typescript
it('test name', () => {
  const data = MockData.*ACTION_NAME_SUCCESS_INDEX*;
  const action = new actions.*Action1Success*(data);
  state = {
    ...state
  };
  const expected = {
    ...state
  };
  expect(reducer(state, action)).toEqual(expected);
});
```
{% endcode %}
{% endtab %}

{% tab title="example" %}
```typescript
it('should handle load all available external premises success correctly', () => {
  const data = MockData.LOAD_ALL_AVAILABLE_EXTERNAL_PREMISES_SUCCESS_1;
  const action = new actions.LoadAllAvailableExternalPremisesSuccess(data);
  const info = {
    hostname: '1.2.3.4',
    loginRequest: {
      username: 'admin',
      password: '12345'
    }
  };
  state = {
    ...state,
    allAvailableExternalPremises: {
      state: ExternalPremisesStates.LOADING,
      error: null,
      data: {
        info: info
      }
    }
  };
  const expected = {
    ...state,
    allAvailableExternalPremises: {
      state: ExternalPremisesStates.DEFAULT,
      error: null,
      data: {
        info: info,
        response: data,
      },
    }
  };
  expect(reducer(state, action)).toEqual(expected);
});
```
{% endtab %}
{% endtabs %}




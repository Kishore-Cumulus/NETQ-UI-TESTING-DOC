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

* writing a unit test. There are  three/four steps in a unit test for selector

  * selecting the selector that to be tested
  * expected value from the selector.
  * de-structure state along with that value
  * passing the state into projector of the selector and asserting against expected value.

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


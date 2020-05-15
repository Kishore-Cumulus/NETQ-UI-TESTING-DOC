# Testing Effects

## Concepts

Let's start with going through one sample effect and think about the components involved and how we can test it.

```typescript
  @Effect()
  loadAllAddedExternalPremises$: Observable<Action> = this.actions$.pipe(
    ofType(actions.LOAD_ALL_ADDED_EXTERNAL_PREMISES),
    switchMap(() => this.externalPremisesApiService.fetchAllAddedExternalPremises()
      .pipe(
        map((externalPremises) => new actions.LoadAllAddedExternalPremisesSuccess(externalPremises)),
        catchError(error => of(new actions.DefaultExternalPremisesFail(error))),
      ),
    ),
  );
```

 There are three important components that make an complete effect

1.  Effect decorator

   ```typescript
   @Effect()
   ```

2. Actions observable

   ```typescript
   this.actions$
   ```

3. Effect implementation. The rest of the things, specifically the part inside the pipe.

Actions observable can be thought of stream of all actions to which the effect implementation listens to and filter them by using the ofType

Effect decorator basically do subscription of this effect implementation and it chains the final action which was returned by effects implementation and dispatch it to the store again.

For testing a effect, we need to create a mock actions observable so that we can next/emit required actions to our need. Inside effect implementation we see a lot of dependencies on api service which basically returns with observables. Thinking about the effect decorator we don't need to do anything for that explicitly because if we can test the effect implementation to return the correct observable at the end, then we assume the effect can dispatch the same to the store\(that would be a different testing case which would be at the effects code\)

For the mock actions observable, ngrx have a helper function `provideMockActions` which takes an action observable and wraps with a provider token and returns a provider

For mocking the api service we can use normal observables similar to the stubbing in the components. There is also other more powerful alternative called `jasmine-marbles`  which rxjs also uses internally for testing their implementations. I recommend you go through the referred links to know more details and more use cases [RxJs writing marble tests](https://github.com/ReactiveX/rxjs/blob/5.4.2/doc/writing-marble-tests.md) and a [shorter version](https://ncjamieson.com/marble-testing-rtfm/)

I will go through the most used parts of it.

```typescript
it('should return a LoadAllAddedExternalPremisesSuccess on success', () => {
  const action = new actions.LoadAllAddedExternalPremises();
  const response = MockData.LOAD_ALL_ADDED_EXTERNAL_PREMISES_SUCCESS_1;
  const expected = new actions.LoadAllAddedExternalPremisesSuccess(response);

  actions$ = hot('-a', {a: action});
  const respones$ = cold('-a|', { a: response });
  const expected$ = cold('--b', { b: expected });
  externalPremisesApiService.fetchAllAddedExternalPremises = jest.fn(() => respones$);

  expect(effects.loadAllAddedExternalPremises$).toBeObservable(expected$);
});
```

Syntax of a normal marble 

```typescript
actions$ = hot('-a', {a: action});
const respones$ = cold('-a|', { a: response });    
```

Taken from the rxjs documentation.

* `"-"` time: 10 "frames" of time passage.
* `"|"` complete: The successful completion of an observable. This is the observable producer signaling `complete()`
* `"#"` error: An error terminating the observable. This is the observable producer signaling `error()`
* `"a"` any character: All other characters represent a value being emitted by the producure signaling `next()`
* `"()"` sync groupings: When multiple events need to single in the same frame synchronously, parenthesis are used to group those events. You can group nexted values, a completion or an error in this manner. The position of the initial `(` determines the time at which its values are emitted.
* `"^"` subscription point: \(hot observables only\) shows the point at which the tested observables will be subscribed to the hot observable. This is the "zero frame" for that observable, every frame before the `^` will be negative.

in our use case, we need to create a actions$ observable if not used jasmine marbles we would have like below

```typescript
const action = new actions.LoadAllAddedExternalPremises();
actions$ = new BehaviourSubject(action).asObservable().delay(10)
```

and this is how if it's marble syntax

```typescript
const action = new actions.LoadAllAddedExternalPremises();
actions$ = hot('-a', {a: action});
```

`BehaviourSubject().asObservable()` is analogous to `hot()`. `delay(10)` is analogous to  ```-``` 

if we use another example 

```typescript
const expected = new actions.LoadAllAddedExternalPremisesSuccess(response);
const expected$ = cold('--b', { b: expected });
```

this is analogous to

```typescript
const expected = new actions.LoadAllAddedExternalPremisesSuccess(response);
const expected$ = Of(expected).delay(20);
```

{% hint style="info" %}
delay is not exactly similar to frame, we can think of one frame as one execution loop
{% endhint %}

## Implementation

Let's start with the an effect which we take as sample for our testing

```typescript
@Effect()
AddExternalPremises$: Observable<Action> = this.actions$.pipe(
  ofType(actions.ADD_EXTERNAL_PREMISES),
  switchMap(({ newExternalPremises } ) => this.externalPremisesApiService.addExternalPremises(newExternalPremises)
    .pipe(
      map((newExternalPremisesRes) => new actions.AddExternalPremisesSuccess(newExternalPremisesRes)),
      catchError(error => of(new actions.DefaultExternalPremisesFail(error))),
    ),
  ),
);
```

#### Steps

1. create a new file beside the effects with a \*testing.effects.spec.ts\* extension with the following content\(copy and paste this\) and replace it with your relevant things wherever it starts with \* and ends with \*

{% tabs %}
{% tab title="structure" %}
{% code title=" store/\*testing.effects.spec.ts\* " %}
```typescript
import { TestBed } from '@angular/core/testing';
import { provideMockActions } from '@ngrx/effects/testing';
import { cold, hot } from 'jasmine-marbles';
import { Observable } from 'rxjs';

import { *TestingApiService* } from '*../services/testing-api.service*';

import * as actions from '*./testing.actions*';
import { *TestingEffects* } from '*./testing.effects*';

describe('ExternalPremisesEffects', () => {
  let actions$: Observable<any>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        provideMockActions(() => actions$),
        *TestingEffects*,
        {
          provide: *TestingApiService*,
          useValue: {
            *testingMethod1*: jest.fn(),
            *testingMethod2*: jest.fn(),
          }
        },
      ]
    });
    effects = TestBed.get(*TestingEffects*);
    *testingApiService* = TestBed.get(*TestingApiService*);
  });

  it('should be created', () => {
    expect(effects).toBeTruthy();
    expect(testingApiService).toBeTruthy();
  });
});

```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="store/external-premises/external-premises.effects.spec.ts" %}
```typescript
import { HttpErrorResponse } from '@angular/common/http';
import { TestBed } from '@angular/core/testing';
import { provideMockActions } from '@ngrx/effects/testing';
import { cold, hot } from 'jasmine-marbles';
import { Observable } from 'rxjs';

import { ExternalPremisesApiService } from '../../services/external-premises/external-premises-api.service';

import * as actions from './external-premises.actions';
import { ExternalPremisesEffects } from './external-premises.effects';
import { MockData } from './mocks/mock-data';

describe('ExternalPremisesEffects', () => {
  let actions$: Observable<any>;
  let effects: ExternalPremisesEffects;
  let externalPremisesApiService:  ExternalPremisesApiService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        provideMockActions(() => actions$),
        ExternalPremisesEffects,
        {
          provide: ExternalPremisesApiService,
          useValue: {
            fetchAllAddedExternalPremises: jest.fn(),
            addExternalPremises: jest.fn(),
            updateExternalPremises: jest.fn(),
            deleteExternalPremises: jest.fn(),
            fetchAllAvailableExternalPremises: jest.fn()
          }
        },
      ]
    });
    effects = TestBed.get(ExternalPremisesEffects);
    externalPremisesApiService = TestBed.get(ExternalPremisesApiService);
  });

  it('should be created', () => {
    expect(effects).toBeTruthy();
    expect(externalPremisesApiService).toBeTruthy();
  });
});

```
{% endcode %}
{% endtab %}
{% endtabs %}

2. create a new folder called mocks with a file mock-data.ts with content as below and import the same into the spec file.

{% tabs %}
{% tab title="structure" %}
{% code title="store/mocks/mock-data.ts" %}
```typescript
export class MockData {
  static *ACTION_NAME_INDEX* = {
  };

  static *ACTION_NAME_SUCCESS_INDEX* = {
  
  };
  static *ACTION_NAME_FAILURE_INDEX* = {
  };
}

```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="store/external-premises/mocks/mock-data.ts" %}
```typescript
export class MockData {
  static ADD_EXTERNAL_PREMISES_1 = {
    hostname: '1.2.3.4',
    id: 12345,
    name: 'premise1',
    creationTime: '29/09/2020'
  };

  static ADD_EXTERNAL_PREMISES_SUCCESS_1 = {
    hostname: '1.2.3.4',
    id: '123456',
    name: 'Premises 1'
  };

  static ADD_EXTERNAL_PREMISES_FAILURE_1 = {
    message: 'External Premises with opid {opid} already exists'
  };
}

```
{% endcode %}
{% endtab %}
{% endtabs %}

3. create a describe block inside the main for every effect since it can have multiple success and failure cases. so the new describe block can be a group. This is not mandatory, feel free to structure as your prefer.

{% tabs %}
{% tab title="structure" %}
{% code title="store/\*testing.effects.spec.ts\* " %}
```typescript
  describe('*TestingEffect1$*', () => {
    it('should return a *Action1Success* on success', () => {
      const data = MockData.*ACTION_NAME_INDEX*;
      const action = new actions.*Action1*(data);
      const response = MockData.*ACTION_NAME_SUCCESS_INDEX*;
      const expected = new actions.*Action1Success*(response);

      actions$ = hot('-a', {a: action});
      const respones$ = cold('-a|', { a: response });
      const expected$ = cold('--b', { b: expected });
      *testingApiService.testingMethod1* = jest.fn(() => respones$);

      expect(effects.*TestingEffect1$*).toBeObservable(expected$);
    });
    it('should return a *Action1Fail* on error', () => {
      const data = *MockData.ACTION_NAME_INDEX*;
      const action = new actions.*Action1*(data);
      const errorData = MockData.*ACTION_NAME_FAILURE_INDEX*;
      const error = new HttpErrorResponse({error: errorData});
      const expected = new actions.*Action2Fail*(error);

      actions$ = hot('-a', {a: action});
      const error$ = cold('-#|', {}, error);
      const expected$ = cold('--b', { b: expected });
      *testingApiService.testingMethod1* = jest.fn(() => error$);

      expect(effects.*TestingEffect1$*).toBeObservable(expected$);
    });
  });
```
{% endcode %}
{% endtab %}

{% tab title="example" %}
{% code title="store/external-premises/external-premises.effects.spec.ts" %}
```typescript
describe('AddExternalPremises$', () => {
  it('should return a AddExternalPremisesSuccess on success', () => {
    const data = MockData.ADD_EXTERNAL_PREMISES_1;
    const action = new actions.AddExternalPremises(data);
    const response = MockData.ADD_EXTERNAL_PREMISES_SUCCESS_1;
    const expected = new actions.AddExternalPremisesSuccess(response);

    actions$ = hot('-a', {a: action});
    const respones$ = cold('-a|', { a: response });
    const expected$ = cold('--b', { b: expected });
    externalPremisesApiService.addExternalPremises = jest.fn(() => respones$);

    expect(effects.AddExternalPremises$).toBeObservable(expected$);
  });
  it('should return a DefaultExternalPremisesFail on error', () => {
    const data = MockData.ADD_EXTERNAL_PREMISES_1;
    const action = new actions.AddExternalPremises(data);
    const errorData = MockData.ADD_EXTERNAL_PREMISES_FAILURE_1;
    const error = new HttpErrorResponse({error: errorData});
    const expected = new actions.DefaultExternalPremisesFail(error);

    actions$ = hot('-a', {a: action});
    const error$ = cold('-#|', {}, error);
    const expected$ = cold('--b', { b: expected });
    externalPremisesApiService.addExternalPremises = jest.fn(() => error$);

    expect(effects.AddExternalPremises$).toBeObservable(expected$);
  });
});
```
{% endcode %}
{% endtab %}
{% endtabs %}


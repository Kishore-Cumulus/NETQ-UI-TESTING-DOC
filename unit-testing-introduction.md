# Unit testing introduction

Unit tests are the easier to test of all the three. Also unit testing can be further categorised into some groups based on their testing strategy. Before that we will go through the anatomy of the spec file of a component.

### Anatomy of spec file

{% tabs %}
{% tab title="Imports" %}
```typescript
import { Component, EventEmitter, Input, NO_ERRORS_SCHEMA,  Output } from '@angular/core';
import { async, fakeAsync, tick, ComponentFixture, TestBed } from '@angular/core/testing';
import { ExternalPremisesStates } from '../../../../../../netq-management/models';
import { ExternalPremisesService } from '../../../../../../netq-management/services/external-premises/external-premises.service';
import { SelectPremisesComponent } from './select-premises.component';

```
{% endtab %}

{% tab title="Stubbing class dependencies" %}
```typescript
const externalPremisesServiceStub = (): Partial<ExternalPremisesService> => ({
  loadAllAvailableExternalPremises: () => ({}),
  selectAllAvailableExternalPremises: () => (of({
    state: ExternalPremisesStates.DEFAULT,
    data: null,
    error: null,
  })),
  selectAllAddedExternalPremises: () => (of({
    state: ExternalPremisesStates.DEFAULT,
    data: null,
    error: null,
  }))
});
```
{% endtab %}

{% tab title="Stubbing template dependencies" %}
```typescript
@Component({
  selector: 'hzn-bulk-inline-form',
  template: `
  <div *ngFor="let form of forms">
    <p>name: {{form.fields[0].value}}</p>
    <p>id: {{form.fields[1].value}}</p>
  </div>
  `
})
class BulkInlineFormStubComponent {
  @Input() forms;
  @Output() saveForm = new EventEmitter();

  constructor() {
  }
}

```
{% endtab %}

{% tab title="Describe block" %}
```typescript
describe('SelectPremisesComponent', () => {
  let component: SelectPremisesComponent;
  let fixture: ComponentFixture<SelectPremisesComponent>;

  beforeEach(async(() => {

    TestBed.configureTestingModule({
      declarations: [
        SelectPremisesComponent,
        BulkInlineFormStubComponent
      ],
      providers: [
        {provide: ExternalPremisesService, useFactory: externalPremisesServiceStub}
      ],
      schemas: [NO_ERRORS_SCHEMA]
    })
      .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(SelectPremisesComponent);
    component = fixture.componentInstance;
    component.data =  {
      hostname: '1.1.1.1',
      username: 'admin',
      password: 'admin'
    };
  });
});
```
{% endtab %}
{% endtabs %}

#### 

{% tabs %}
{% tab title="test block" %}
```typescript
  it('should create', () => {
    fixture.detectChanges();
    expect(component).toBeTruthy();
  });
```
{% endtab %}

{% tab title="test block using snapshot" %}
```typescript
it('should show error state ', fakeAsync(() => {
  const externalPremisesServiceStub = fixture.debugElement.injector.get(ExternalPremisesService);
  externalPremisesServiceStub.selectAllAvailableExternalPremises = () => (of({
    state: ExternalPremisesStates.DEFAULT,
    data: null,
    error: {error : {message: 'I am the error'}},
  })).pipe(delay(2000));
  fixture.detectChanges();
  tick(2000);
  fixture.detectChanges();
  expect(fixture).toMatchSnapshot();
}));
  
// generated snapshot inside __snapshots__ folder  
exports[`SelectPremisesComponent should show error state  1`] = `
<hzn-select-premises
  data={[Function Object]}
  externalPremisesService={[Function Object]}
  formData={[Function EventEmitter]}
  handleAdd={[Function Function]}
  handleSave={[Function Function]}
  selectAllAvailableExternalPremises$={[Function Observable]}
  setValidationMessage={[Function Function]}
  subscription$={[Function Subscriber]}
  transformToList={[Function Function]}
>
  <div
    class="info-wrapper"
    title="I am the error"
  >
    <span
      class="svgicon error"
    />
    <span
      class="message"
    >
      I am the error
    </span>
  </div>
</hzn-select-premises>
`;
  
  
```
{% endtab %}
{% endtabs %}

There are four main blocks of code inside a spec file. 

1. **Imports**: Contains the default testing helper modules, component to test, dependencies of the component
2.  **Stubs**: There are majorly two types of  stubs 
   1. Class stubs\(services which are injected into the components\)
   2. Template stubs\(component's template dependencies\)
3. **Describe**: it is the container block which holds the `before`, `beforeEach`, `afterEach`, `after` hooks and the test blocks. It will have component initialisation logics like preparing the testing module with our testing component and stubbed dependencies. This module is similar to the normal module but gives us functionality on creating components and running the lifecycle hooks.
4. **Test blocks:**  individual test cases

{% hint style="info" %}
We can always test without stubs, but it eventually means that we are depending on the external components to work properly for our tests to work and in the case of services which can cause side-effects like API calls. And to mimic multiple cases it's better to use stubs.

One way of not using template stubs is to give `NO_ERRORS_SCHEMA` in providers of `configureTestingModule` so that angular won't complain unrecognised elements in the component.
{% endhint %}



### Unit test types

1. Isolated tests
2. Shallow tests

Both the above test can also be used with snapshot testing which can ease the test cases and decreases the amount of code written in the test case\(we will discuss it in the later parts\).

#### Isolated tests

In the context of Angular components, isolated test cases can be thought of testing the functionality of the code in .ts/class files without considering the template. We don't need template stubs and angularTestingModule in isolated test case we can just create a new object similar to a normal JS/TS class and tests it's functionality. We can use this sort of testing when we want to make sure that the internal methods are working fine.

Pipes and most of the redux store testing like the reducers, selectors, effects we use the isolated test case approach.

Note that even in case of isolated test cases, if a component is depending on some services in it's constructor we need to mock those service called class stubs.

**Shallow tests**

In many of the cases when testing for angular components we need to test along with the templates and the interaction between the class and template. For this type of testing as shown above angular provides `configureTestingModule` where we can create a complete testing component.

we use this approach for all the components and for directives




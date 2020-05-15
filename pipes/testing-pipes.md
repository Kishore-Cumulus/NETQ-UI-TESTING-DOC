# Testing Pipes

 Testing pipes is more easier than component as they don't have a template and are more simpler and straight forward. We can test Pipes in two ways 

1. Testing the pipe class directly by calling their transform method from the test cases.
2. Creating a host component which uses this pipe. This is more realistic since we are testing the pipe on the basis of how it's used. This same pattern of testing using host component can be used with directives and components also.

We will go through both the ways 

Let's take the active-item pipe as an example

{% tabs %}
{% tab title="active-items.pipe.spec.ts" %}
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'activeItems',
})
export class ActiveItemsPipe implements PipeTransform {
  transform(items: Object[], prop: string, isActive: boolean = true): Object[] {
    return items ? items.filter(item => isActive ? !item[prop] : item[prop]) : [];
  }
}

```
{% endtab %}

{% tab title="Mock data" %}
```typescript
class MockData {
  static ITEMS_1 = [
    {
      id: 1,
      isValid: false,
    },
    {
      id: 2,
      isValid: true,
    },
    {
      id: 3,
      isValid: true,
    },
    {
      id: 4,
      isValid: false,
    },
    {
      id: 5,
      isValid: true,
    },
  ];
}
```
{% endtab %}
{% endtabs %}

#### Testing the pipe class directly

```bash
import { async, ComponentFixture, TestBed } from '@angular/core/testing';

import { ActiveItemsPipe } from './active-items.pipe';


describe('ActiveItemsPipe', () => {

  beforeEach();

  it('filter items for falsy prop value for default isActive', () => {
    const items = MockData.ITEMS_1;
    const prop = 'isValid';
    const pipe = new ActiveItemsPipe();
    const transformed = pipe.transform(items, prop);
    expect(transformed.length).toEqual(2);
  });
});

```

In this case we are importing the pipe and creating a new instance and asserting on it's transform. In this case we are not concerned how it will be used but the more important thing is the functionality to be working. 

#### Testing the pipe using a host component

Let's write the same test using a host component technique

```bash
import { Component } from '@angular/core';
import { async, ComponentFixture, TestBed } from '@angular/core/testing';

import { ActiveItemsPipe } from './active-items.pipe';


@Component({
  selector: 'hzn-host',
  template: `
    <div class="item" *ngFor="let item of (items | activeItems : 'isValid' : isActiveProp)">
      <p>{{item.id}}</p>
    </div>
  `
})
class HostComponent {
  items;
  isActiveProp = true;
  constructor() {}
}


describe('ActiveItemsPipe', () => {


  let component: HostComponent;
  let fixture: ComponentFixture<any>;

  beforeEach(async(() => {

    TestBed.configureTestingModule({
      declarations: [
        ActiveItemsPipe,
        HostComponent,
      ]
    }).compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(HostComponent);
    component = fixture.componentInstance;
  });


  it('filter items for falsy prop value for default isActive', () => {
    component.items = MockData.ITEMS_1;
    fixture.detectChanges();
    const componentEl: HTMLElement = fixture.nativeElement;
    const itemsList = componentEl.querySelectorAll('.item');
    expect(itemsList.length).toEqual(2);
    expect(fixture).toMatchSnapshot();
  });
});

```

In this type of tests, the pipe is tested in the way how it will be used in real world. We create Host component which has the basic functionality where the pipe can be used. One advantage is that by using the snapshot we will get know which will be rendered.

```bash
exports[`ActiveItemsPipe filter items for falsy prop value for default isActive 1`] = `
<hzn-host
  isActiveProp={[Function Boolean]}
  items={[Function Array]}
>
  <div
    class="item"
  >
    <p>
      1
    </p>
  </div><div
    class="item"
  >
    <p>
      4
    </p>
  </div>
</hzn-host>
`;
```

{% hint style="info" %}
Snapshots can be generated over normal objects\(first case\) but not limited to fixtures. We need to add serialisers for the same. Refer to jest snapshot serialisers for more details.
{% endhint %}


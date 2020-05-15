# Snapshot testing

You can write your assertions in a couple of ways. One way is the most common way like to query the nativeElement and asserting against that like the below

```typescript
  const hostElement = fixture.nativeElement;
  const nameInput: HTMLInputElement = hostElement.querySelector('input');
  const nameDisplay: HTMLElement = hostElement.querySelector('span');
  nameInput.value = 'quick BROWN  fOx';
  nameInput.dispatchEvent(newEvent('input'));

  // Tell Angular to update the display binding through the title pipe
  fixture.detectChanges();

  expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
```

and since we are using jest, it gives us a more intuitive way of assertions

```typescript
expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
// with
expect(fixture).toMatchSnapshot();
```

if this test runs for the first time toMatchSnapshot\(\) creates a snapshot of the template at that state and saves it inside \_\_snapshots\_\_ folder relative to the spec file. For the first time we need to verify the snap file and check whether the content of it is valid.  Sample snapshot below.

```text
exports[`SelectPremisesComponent should filter out already added premises in the list 1`] = `
<hzn-select-premises
  allAvailablePremises={[Function Array]}
  allAvailablePremisesForms={[Function Array]}
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
    class="inline-form-wrapper"
  >
    <hzn-bulk-inline-form>
      
      <div>
        <p>
          name: premise1
        </p>
        <p>
          id: 12345
        </p>
      </div>
    </hzn-bulk-inline-form>
  </div>
</hzn-select-premises>
`;
```

From the further runs, jest picks the created snapshot and compares it to the result. If there is any mismatch the assertion will be failed and we would be getting a diff of the comparison similar to git.

 

{% hint style="info" %}
Snapshot testing can result in lesser code in the test case and is more intuitive for frontend purpose.
{% endhint %}

Also we can also combine both these assertions inside the test. The below example has both the normal assertion and a snapshotMatch assertion. This is a completely valid test case. 

{% code title="src/app/modules/shared/pipes/active-items/active-items.pipe.spec.ts" %}
```typescript
  it('filter items for falsy prop value for default isActive', () => {
    component.items = MockData.ITEMS_1;
    fixture.detectChanges();
    const componentEl: HTMLElement = fixture.nativeElement;
    const itemsList = componentEl.querySelectorAll('.item');
    expect(itemsList.length).toEqual(2);
    expect(fixture).toMatchSnapshot();
  });
```
{% endcode %}


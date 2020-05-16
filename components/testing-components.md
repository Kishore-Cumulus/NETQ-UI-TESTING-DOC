# Testing Components

## Jump start

Replace the content in the spec file with the content below and make changes at the \*...\* with your relevant modules.

```typescript
import { Component, EventEmitter, Input, NO_ERRORS_SCHEMA,  Output } from '@angular/core';
import { async, fakeAsync, tick, ComponentFixture, TestBed } from '@angular/core/testing';
import { of } from 'rxjs';
import { delay } from 'rxjs/operators';

import { *TestingService* } from '*../testing.service*';

import { *TestingComponent* } from './testing.component';


// use this part to stub any dependent templates.
@Component({
  selector: '*hzn-some-other*',
  template: `
  <div *ngFor="let form of forms">
    <p>name: {{form.fields[0].value}}</p>
    <p>id: {{form.fields[1].value}}</p>
  </div>
  `
})
class *SomeOtherStubComponent* {
  @Input() forms;
  @Output() saveForm = new EventEmitter();

  constructor() {
  }
}

// use this to stub any pipe in the template
@Pipe({
  name: '*some-other*'
})
export class *SomeOtherStubPipe* implements PipeTransform {
  transform() {
  }
}

describe('*TestingComponent*', () => {
  let component: TestingComponent;
  let fixture: ComponentFixture<*TestingComponent*>;

  beforeEach(async(() => {

    const *testingServiceStub* = (): Partial<*TestingService*> => ({
      *loadMethod1*: () => ({}),
      *selectMethod1*: () => (of()),
    });

    TestBed.configureTestingModule({
      declarations: [
        *TestingComponent*,
        *SomeOtherStubComponent*,
        *SomeOtherStubPipe*
      ],
      providers: [
        {
          provide: *TestingService*, 
          useFactory: *testingServiceStub*
        }
      ],
      schemas: [NO_ERRORS_SCHEMA]
    })
      .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(*TestingComponent*);
    component = fixture.componentInstance;
    // if any input props
    component.*inputProp1* =  {};
  });

  it('should create', () => {
    fixture.detectChanges();
    expect(component).toBeTruthy();
  });

  it('should *test some async func*', fakeAsync(() => {
    const *testingServiceStub* = fixture.debugElement.injector.get(*TestingService*);
    testingServiceStub.selectMethod1 = () => (of(*some other data*));
    fixture.detectChanges();
    tick();
    expect(fixture).toMatchSnapshot();
  }));

  it('should *test some async func with a delay*', fakeAsync(() => {
    const *testingServiceStub* = fixture.debugElement.injector.get(*TestingService*);
    testingServiceStub.selectMethod1 = () => (of(*some other data*)).pipe(delay(2000));
    fixture.detectChanges();
    tick(2000);
    fixture.detectChanges();
    expect(fixture).toMatchSnapshot();
  }));
  
  it('should *test some other input props*', fakeAsync(() => {
    component.*inputProp1* =  *some other data*;
    const *testingServiceStub* = fixture.debugElement.injector.get(*TestingService*);
    testingServiceStub.selectMethod1 = () => (of(*some other data*));
    fixture.detectChanges();
    expect(fixture).toMatchSnapshot();
  }));
  
  it('should *test some template user interaction*', () => {
    // get the name's input and display elements from the DOM
    const hostElement = fixture.nativeElement;
    const nameInput: HTMLInputElement = hostElement.querySelector('input');
    const nameDisplay: HTMLElement = hostElement.querySelector('span');
    nameInput.value = 'quick BROWN  fOx';
    nameInput.dispatchEvent(newEvent('input'));
    fixture.detectChanges();
    expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
  });
});

```

## Concept

Let's take an existing component and try writing unit test cases 

{% tabs %}
{% tab title="add-image.component.ts" %}
{% code title="src/app/modules/shared/components/modals/add-image/add-image.component.ts" %}
```typescript
import { Component, Inject, OnDestroy, OnInit } from '@angular/core';
import { KuiOverlayRef, KUI_DIALOG_DATA } from '@kleeen/components';
import { Observable } from 'rxjs';

import { first } from '../../../utils/common.utils';

import { Image } from './../../../../lcm/models';
import { LCMService } from './../../../../lcm/services/lcm/lcm.service';
@Component({
  selector: 'hzn-add-image',
  templateUrl: './add-image.component.html',
  styleUrls: ['./add-image.component.scss']
})
export class AddImageComponent implements OnInit, OnDestroy {

  files: string[] = [];
  selectedImage: File;
  progress$: Observable<number>;
  newImage$: Observable<Image>;
  error$: Observable<any>;
  constructor(
    @Inject(KUI_DIALOG_DATA) public data: any,
    public dialogRef: KuiOverlayRef,
    public lcmService: LCMService
  ) { }

  ngOnInit() {
    this.progress$ = this.lcmService.selectProgress();
    this.newImage$ = this.lcmService.selectNewImage();
    this.error$ = this.lcmService.selectImageUploadError();
  }

  public onCancel() {
    this.dialogRef.close();
  }

  public importImage() {
    if (this.selectedImage) {
      this.lcmService.uploadImage(this.selectedImage);
    }
  }

  public uploadFile(event) {
    this.selectedImage = first(event);
    this.files.push(this.selectedImage.name);
  }
  ngOnDestroy() {
    this.lcmService.resetImageError();
    this.lcmService.resetNewImage();
    this.lcmService.resetProgress();
  }
}

```
{% endcode %}
{% endtab %}

{% tab title="add-image.component.html" %}
{% code title="src/app/modules/shared/components/modals/add-image/add-image.component.html" %}
```markup
<section class="modal import-image">
  <header >
    <span class="title">
      Import Image
    </span>
    <button class="close" (click)="onCancel()" [disabled] = '( (progress$ | async) !== null && (progress$ | async) < 100  && ((error$ | async) === null))'>
      <kui-icon kuiSize="xl" kuiIcon="close"></kui-icon>
    </button>
  </header>
  <div class="import-block"  *ngIf= '((error$ | async) === null) && (((progress$ | async) === null) || (progress$ | async) <= 100) && ((newImage$ | async) === null)'>
    <div class="uploadfilecontainer" (click)="fileInput.click()" hznDragDrop *ngIf='files.length === 0' (fileDropped)="uploadFile($event)">
      <kui-icon kuiIcon="download-bottom" kuiSize="size-3xl"></kui-icon>
      <p class="row">Drag and Drop Image here to Upload</p>
      <p>Or</p>
      <kui-button kuiStyle="primary button-large">
        Browse Image
      </kui-button>
      <input hidden type="file" accept=".bin" #fileInput (change)="uploadFile($event.target.files)">
    </div>

    <div  class="files-list" *ngFor="let file of files;let i=index">
      <p>{{ file }}</p>
    </div>
  </div>

  <div class="progress" *ngIf= '((error$ | async) === null) && ((progress$ | async) !== null && (progress$ | async) <= 100) && ((newImage$ | async) === null)'>
    <div class="progress-bar">
      <span
        class="completed"
        [style.width.%]='(progress$ | async)'
      ></span>
    </div>
    <p class="message">Importing Image ({{(progress$ | async)}}%)</p>
  </div>

  <div class="import-completed" *ngIf= '((error$ | async) === null) && (newImage$ | async) '>
    <kui-icon kuiIcon="check-circle" kuiSize="size-3xl"></kui-icon>
    <p class="message">Image Uploaded Successfully</p>
    <div class="image-details">
      <ul>
        <li class="info"><span class="title">Name : </span>{{ (newImage$ | async).file_name }}</li>
        <li class="info"><span class="title">ASIC Vendor : </span>{{ (newImage$ | async).asic_vendor }}</li>
        <li class="info"><span class="title">CL Version : </span>{{ (newImage$ | async).cl_version }}</li>
        <li class="info"><span class="title">CPU Arch : </span>{{ (newImage$ | async).cpu_arch }}</li>
      </ul>
    </div>
    <footer class="wizard-footer">
      <div class="wizard-controls-wrapper">
        <ul class="wizard-controls">
          <li>
            <kui-button
              (click) = 'onCancel()'
              kuiStyle="primary button-medium"
            >
              Done
            </kui-button>
          </li>
        </ul>
      </div>
    </footer>
  </div>

  <div class="error-wrapper" *ngIf= '(error$ | async) !== null' [title]="error$ | async">
      <span class="icon error" [inlineSVG]="'../../../../../../assets/icons/light/warn.svg'"></span>
      <span class="error-title">{{ 'LCM.IMAGES.UPLOAD_FAIL' | translate}}</span>
      <span class="error-message">{{ error$ | async }}</span>
  </div>

  <footer class="wizard-footer" *ngIf= '((progress$ | async) === null) && ((error$ | async) === null)'>
    <div class="wizard-controls-wrapper">
      <ul class="wizard-controls">
        <li>
          <kui-button
            (click) = 'importImage()'
            kuiStyle="primary button-medium"
          >
            Import
          </kui-button>
        </li>
      </ul>
    </div>
  </footer>
</section>

```
{% endcode %}
{% endtab %}
{% endtabs %}

When testing a component, you would highly likely to want to run the test cases of that specific file rather than running all the unit tests for all the components. There are couple of ways one way is to pass regex pattern to `npm run test` and other way is to use jest with the filename like:

```bash
npx jest src/app/modules/shared/components/modals/add-image/add-image.component.spec.ts
```

I find this command more easier to use and also to remember. Also please go with your own preference.

In most of cases if we run the above command on the default spec file you would be getting errors saying  

{% hint style="danger" %}
```text
Template parse errors:
'kui-icon' is not a known element:
1. If 'kui-icon' is an Angular component, then verify that it is part of this module.
2. If 'kui-icon' is a Web Component then add 'CUSTOM_ELEMENTS_SCHEMA' to the '@NgModule.schemas' of this component to suppress this message. (" (progress$ | async) !== null && (progress$ | async) < 100  && ((error$ | async) === null))'>
      [ERROR ->]<kui-icon kuiSize="xl" kuiIcon="close"></kui-icon>
```
{% endhint %}

because in most cases some sort of dependencies inside the template like kui-icon, kui-button any custom angular components which angular can't identify without adding them to the declarations. You may think that as these dependencies are added in component NgModule it should not cause any issue. But the important thing to note is the NgModule where this component is declared is of no use in the spec file, in fact in the spec file we create a new NgModule using configureTestingModule  and this is where you need to add the dependent declarations as you add in the normal module.

```typescript
  TestBed.configureTestingModule({
    declarations: [
      AddImageComponent,
    ],
    providers: [
    ],
    schemas: [NO_ERRORS_SCHEMA]
  })
    .compileComponents();
  }));
```

1. Add **`NO_ERRORS_SCHEMA`** to the **`schemas`** array of the **`configureTestingModule`**. this will suppress all the undeclared component errors in the template.
2. After adding this if your component uses any pipes, in many cases we will be using, we need to stub those pipes. I agree this is pain stacking for the first time, but after we write some more spec files we can copy paste those required stubs. 

```typescript
@Pipe({
  name: 'translate'
})
export class TranslatePipeStub implements PipeTransform {
  transform() {
  }
}
```

and add these stubs to **`declarations`** array.

3. All the template errors by this time should be resolved. by using _**`NO_ERRORS_SCHEMA`**_ all the undeclared components warnings are suppressed and angular won't compile that component. But if you want to render some dependent template like say `hzn-bulk-inline-form` we can stub a basic implementation of it like below and add it to the declarations.

{% code title="shared/components/modals/premise-dialog/component/select-premises/select-premises.component.spec.ts" %}
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
```
{% endcode %}

4. Now try running the command again, we will be getting class dependency errors. it's time to mock the class dependencies. if you see the class constructor there are three dependencies and we will stub them with minimal implementation.

```typescript
  beforeEach(async(() => {

    const kuiDialogDataStub = () => ({});
    const kuiOverlayRefStub = () => ({
      close: () => ({}),
    });
    const lcmServiceStub = (): Partial<LCMService> => ({
      selectProgress: () => (of(0)),
      selectNewImage: () => {
        let image: Image;
        return of(image);
      },
      selectImageUploadError: () => {
        return of('');
      }
    });

    TestBed.configureTestingModule({
      declarations: [
        TranslatePipeStub,
        AddImageComponent,
      ],
      providers: [
        {
          provide: KUI_DIALOG_DATA,
          useFactory: kuiDialogDataStub,
        },
        {
          provide: KuiOverlayRef,
          useFactory: kuiOverlayRefStub,
        },
        {
          provide: LCMService,
          useFactory: lcmServiceStub,
        }
      ],
      schemas: [NO_ERRORS_SCHEMA]
    })
      .compileComponents();
  }));
  
  
```

5. now running the test command the default test case of component to be defined will be passed. We can now write specific tests and also modify the function behaviour of the stubbed services at the test level specific to the context of the test.

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
```

6. You can also use jest.spyOn and jest.mock to change the stubbing behaviour. Feel free to go through them and use if you feel that it's suited better for your use case.

7. Also when using timer, intervals inside a test case as used above, you need to wrap your test code inside a `fakeAsync` so that you can control the timer using `tick`

8. One more thing I recommend is to remove the fixture.detectChanges into the every test case because it would give you can have control on the lifecycle completely.

{% tabs %}
{% tab title="before" %}
```typescript
  beforeEach(() => {
    fixture = TestBed.createComponent(AddImageComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
```
{% endtab %}

{% tab title="after" %}
```
  beforeEach(() => {
    fixture = TestBed.createComponent(AddImageComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    fixture.detectChanges();
    expect(component).toBeTruthy();
  });
```
{% endtab %}
{% endtabs %}

9. User interactions can be mimicked by using `dispatchEvent`

```typescript
it('should convert hero name to Title Case', () => {
  // get the name's input and display elements from the DOM
  const hostElement = fixture.nativeElement;
  const nameInput: HTMLInputElement = hostElement.querySelector('input');
  const nameDisplay: HTMLElement = hostElement.querySelector('span');

  // simulate user entering a new name into the input box
  nameInput.value = 'quick BROWN  fOx';

  // dispatch a DOM event so that Angular learns of input value change.
  // use newEvent utility function (not provided by Angular) for better browser compatibility
  nameInput.dispatchEvent(newEvent('input'));

  // Tell Angular to update the display binding through the title pipe
  fixture.detectChanges();

  expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
});
```


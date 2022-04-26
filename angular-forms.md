Let's build a form using current ILR building blocks.
```html
<!-- Simplified from example of creating new templates functggionality  -->
<div>
    <fieldcontrol-textbox [FieldControl]="ProgrammeTitle"/>
    <fieldcontrol-textbox [FieldControl]="AimType"/>
    <fieldcontrol-textbox [FieldControl]="LearnAimRef"/>
    <fieldcontrol-textbox [FieldControl]="FundModel"/>
    <fieldcontrol-textbox [FieldControl]="ProgType"/>
    <fieldcontrol-textbox [FieldControl]="FworkCode"/>
    <fieldcontrol-textbox [FieldControl]="PwayCode"/>
    <fieldcontrol-textbox [FieldControl]="StdCode"/>
    <div class="footer">
        <button type="button" (click)="onSubmit()">Save</button>
    </div>
</div>
```
Looks quite clean, isn't it? As I'm not able to pass `FormGroup`, becouse `FieldcontrolTextbox` uses `[FieldControl]` input with custom interface (yes it isn't FormControl), my options are very limited. 
For example, to validate my form only I can do is to check each field for changes separately and deteminate validation state depending on each field value.
```html
<!-- I would have come up with something like that: -->
<button type="button" (click)="onSubmit()" [disable]="isFormInvalid">Save</button>
```
```ts
get isFormInvalid(): {
    ProgrammeTitle.formControl.invalid ||
    AimType.formControl.invalid ||
    LearnAimRef.formControl.invalid ||
    FundModel.formControl.invalid ||
    ProgType.formControl.invalid ||
    FworkCode.formControl.invalid ||
    PwayCode.formControl.invalid ||
    StdCode.formControl.invalid;
}
```
looks good but it's very slow, we can do something with observables to fix that. Set `isFormInvalid` as a property then listen to each of the field state changes to set that up but that's overkill.
Let's take a look how `onSubmit()` function could look like:
```ts
public onSubmit(): void {
    if (this.isFormInvalid) {
        return;
    }
    
    const formData = {
        programmeTitle: this.ProgrammeTitle.formControl.value,
        aimType: this.AimType.formControl.value,
        learnAimRef: this.LearnAimRef.formControl.value,
        fundModel: this.FundModel.formControl.value,
        progType: this.ProgType.formControl.value,
        fworkCode: this.FworkCode.formControl.value,
        pwayCode: this.PwayCode.formControl.value,
        stdCode: this.stdCode.formControl.value,
    }
    
    this.service.save(formData).subscribe(...);
}
```
That kind of `accidental complexity` will have to be added to each component that uses forms which means almost every component in ILR and this is just simple example, without managment of a `touched`, `dirty` state and many others.
There is also output `(OnFieldChanged)`  it has different behaviour for each of input components class (Probably around 6 different implementations) and plenty different undocumented component bindings, but I'm not going in to that.
Now we can imagine how it increasses amount of code for writing code that is already developed by Angular Team. 

Let's try to do the same thing without using anything that isn't available in angular natively: 
```html
<!-- Simplified from example of existing new templates functggionality  -->
<form [formGroup]="myFormGroup" (ngSubmit)="onSubmit($event)">
    <input formControlName="ProgrammeTitle"/>
    <input formControlName="AimType"/>
    <input formControlName="LearnAimRef"/>
    <input formControlName="FundModel"/>
    <input formControlName="ProgType"/>
    <input formControlName="FworkCode"/>
    <input formControlName="PwayCode"/>
    <input formControlName="StdCode"/>
    <div class="footer">
        <button [disabled]="myFormGroup.invalid">Save</button>
    </div>
</form>
```
and `.ts`
```ts
public onSubmit(): void {
    if (this.myFormGroup.invalid) {
        return;
    }
    
    this.service.save(myFormGroup.value).subscribe(...);
}
```
much simpler (3 times less code to read when debuging and **testing**), forms validation and data collection is handled by AngularForms so we can concentrate on actual bussines logic. 

The final solution for this is to go step by step go thrugh each form we have in ILR and replace them with much simpler and flexible pure Material Angular Modules or develop own components following approach taken to build `InputComponment` which supports Reactive and Template Driven Forms. Remove all mentioned components due to amount of issues identified.

Check for more:
https://angular.io/guide/forms-overview
# ILR Technical Debt and Code Smells

##  Overview

### What is this about?
This file describes problems identified in ILR Web Application and what consequences we may issue having them unresolved. 

### Motivation
Continuously raising technical debt and a lot of code smells identified tempted me to write this down. Hopefully, the document will help people involved in product development to plan how to prioritise and resolve current and rising problems with the software.

### Problem categories
Each issue will have one or more problem categories assigned to it.

- Accessibility - all issuers related to WCAG.
- Performance - everything that could lead to problems with application speed.
- Maintainability - issues associated with maintainability are often named “code smells” and "anti-patterns".
- Reliability - this is commonly referred to as potential bugs or as code that will not have the expected behavior at runtime.
- Security - this is commonly referred to as vulnerabilities or flaws in programs that can lead to the use of the application in a different way than it was designed for.

## Architecture
### Angular Modules
![Maintainablity](https://img.shields.io/badge/Maintainablity-1.svg)
![Performance](https://img.shields.io/badge/Performance-grey.svg)

At the moment we don't use modules so much and as this is the one of Angular fundamentals I think we should take a look on the current situation and think about what can be done in that area. Consequences that come from not having properly structured application are very serious due to high risk of dealing with [Big Ball of Mud](#big-ball-of-mud) antipattern.

Problems:
- Tight coupoling, possible `Big Ball of Mud`.
- Loosing ablity of using angular features such as `CanLoad`, `Lazy loading` or `Preloading`. 
- Application bundle size - we need to load etrie app when user access to our application.
- Fragility - Resistance of refactoring due to high regression risk.

<details>
<summary>Show solution</summary>
So let's have look on current ILR folder structure:

```
components/
├─ administration/
│  ├─ ...
constants/
create-aim-template-page/
developer-playground/
directives/
enums/
extensions/
generics/
guards/
models/
pipes/
services/
widgets/
```

that is content of an `app` folder, there is also *AppModule*, *AppComponent* and *AppRoutingModule* in that folder but that    is something we can skip for now. It doasn't look bad? 
Let's assume I need to create new section in admin panel.
```bash
components/
├─ administration/
│  ├─ new-section/
│  │  ├─ new-section-list/
│  │  ├─ new-section-item/
│  │  ├─ new-section-form/
directives/
├─ new-section.directive.ts
pipes/
├─ new-section.pipe.ts
services/
├─ new-section.service.ts
models/
├─ new-section.model.ts
constants/
├─ new-section.constants.ts

# .spec.ts and few others are skiped to simplify
```
Looks good, isn't it? However I don't want pipe, service or directive releated to new-section to be used by any other section   as it will introduce coupoling between this 2 sections. What I can do to prevent that? I can modify my file structure.
```bash
components/
├─ administration/
│  ├─ new-section/
│  │  ├─ new-section-list/
│  │  ├─ new-section-item/
│  │  ├─ new-section-form/
│  │  ├─ new-section.ts # following angular docs we don't add type for models
│  │  ├─ new-section.directive.ts
│  │  ├─ new-section.pipe.ts
│  │  ├─ new-section.service.ts
```
Now looks way better, but still nothing stops me of using pipe and directive in other sections they just changed their  location but still need to be imported in to app module. To avoid that I can create `Domain Module` inside my new section and    pull it out from `components` folder as it's no longer set of componets due to other angular building blocks. 
```bash
modules/
├─ administration/
├─ new-section/
│  ├─ new-section-list/
│  ├─ new-section-item/
│  ├─ new-section-form/
│  ├─ new-section.ts
│  ├─ new-section.directive.ts
│  ├─ new-section.pipe.ts
│  ├─ new-section.service.ts
│  ├─ new-section.module.ts # new
│  ├─ new-section-routing.module.ts # new
 # In my opinion there is no need to nest modules structure, as it will only make modules hard to identify and could    introduce coupoling between them. If we need to see how module structure look like we can go and check routing modules from    App to Children level.
```
Now seems to be perfect. We have proper `SoC` in place, we can `lazy load` our domain module ([Angular Routing] (#angular-routing)). No need to worry about modifing building blocks of this module and impacing other modules. If your  module grows so it's hard navigate through it you can consider splitting it in to smaller modules. For example.:     `AdministrationModule` will have more than one module inside. You can also create folders for each building block like:     `components, directives, services, guards` but that isn't common thing to see.

 How about if we wan't to share some functionality between modules? There are few solutions for that.
 1. SharedModule - where we keep all components, directives and pipes that will be shared accross different modules.
 2. WidgedModule - I like to call it `Matrerial Angular` approach, where every even small pice has it's own module, which   makes it very flexible as you can import each piece only when you need it. It reduces amount of mocking needed when testing   and also risk of high couopoling.
 3. Angular Libraries - It allows you to create more than one angular project inside one repository.
 4. npm pacakge - eg.: Angular Materia, NgBoostrap.

All four methods can be easliy combined.

Let's take a look on possible final structure, which can be easy extended without increassing application complexity:
```
app/
├─ pages/
│  ├─ dashboard/
|  │  ├─ ...
|  │  ├─ dashboard.module.ts
│  ├─ learner/
|  │  ├─ ...
|  │  ├─ learner.module.ts
│  ├─ administration/
|  │  ├─ ...
|  │  ├─ administration.module.ts
├─ shared/
│  ├─ button/
|  │  ├─ ...
|  │  ├─ button.module.ts
│  ├─ grid/
│  ├─ input/
│  ├─ common/
├─ core/
│  ├─ services/
|  │  ├─ ...
|  │  ├─ authentication.service.ts
|  │  ├─ setting.service.ts
│  ├─ core.module.ts
```
    
Check for more:
- https://angular.io/guide/ngmodules
- https://angular.io/guide/libraries
</details>


### Angular Routing
![Performance](https://img.shields.io/badge/Performance-1.svg)
![Maintainablity](https://img.shields.io/badge/Maintainablity-grey.svg)
![Security](https://img.shields.io/badge/Security-grey.svg)

Problems:
- Loosing ablity of using angular features such as `Resolvers`, `canActivateChild`, `shared templates` and many others listed in [Angular Modules](#angular-modules).
- Route tree is flat which means that child route will never know about parent existance, core conceptual problem.
- Top level module holds information about low level modules routing and configuration.

<details>
<summary>Show Solution</summary>
Current routing strategy in ILR seems to be ok. It's easy to read, but let's take a look closer:
```typescript
// app-routing.module.ts
const routes: Routes = [
     { path: '', redirectTo: '/learners', pathMatch: 'full' },
    { path: 'dashboard', component: DashboardComponent },

    { path: 'learners', component: LearnerRecordsComponent },
    { path: 'learners/learner', component: LearnerComponent },
    { path: 'learners/learner/:learnerId', component: LearnerComponent },

    { path: 'reports', component: ReportsComponent },

    { path: 'providers', component: ProviderRecordsComponent, canActivate: [AuthGuard] },
    { path: 'providers/provider/new', component: ProviderNewComponent, canActivate: [AuthGuard] },
    { path: 'providers/provider/:providerId', component: ProviderViewEditComponent, canActivate: [AuthGuard] },

    { path: 'system-events', component: SystemEventRecordsComponent, canActivate: [AuthGuard] },
    { path: 'system-events/system-event/:systemEventId', component: SystemEventViewComponent, canActivate: [AuthGuard] },

    { path: 'system-mimic', component: SystemMimicComponent, canActivate: [AuthGuard] },

    { path: 'users', component: UserRecordsComponent, canActivate: [AuthGuard] },
    { path: 'users/user/new', component: UserNewComponent, canActivate: [AuthGuard] },
    { path: 'users/user/:userId', component: UserViewEditComponent, canActivate: [AuthGuard] },

    { path: 'admin', component: AdminPageComponent, canActivate: [AuthGuard] },
    { path: 'admin/provider', component: ProviderViewEditComponent, canActivate: [AuthGuard] },
    { path: 'admin/ilr-management', component: IlrManagementComponent, canActivate: [AuthGuard] },
    { path: 'admin/import-learners', component: ImportLearnersComponent, canActivate: [AuthGuard] },

    { path: '**', component: LoggingInPleaseWaitComponent }
    ...
```
So at first point, what we can see it's `AuthGuard` repeated for each child route. We can twak it it like that:
```ts
// app-routing.module.ts
{ 
    path: 'providers',
    canActivate: [AuthGuard] 
    canActivateChild: [AuthGuard],
    children: [
        { path: '', component: ProviderRecordsComponent },
        { path: 'new', component: ProviderNewComponent },
        { path: ':providerId', component: ProviderViewEditComponent },
    ]
},
```
It may sound like I just make it less readable but opens a lot of different opportunities. Now we can resolve data using `Resolvers` for the parent and access that data from `Router` context. This will reduce amount of requests made to an API and reduce complexity of managing data from components so they can focus more on their use cases. As mentioned in [Angular Modules](#angular-modules) section, we can separate entrie providers context to it's own module and by having this routing strucure we can `lazy load` (load a specific module bundle from the server only when user navigate to that page or using `preload strategy`) `Providers` module like that:
```ts
{ 
  path: 'providers',
  canActivate: [AuthGuard] 
  canActivateChild: [AuthGuard],
  loadChildren: () => import('./providers/providers.module').then(m => m.ProvidersModule)
},
```
this will make `AppRouiting` only exposing main routing configuration and will let `ProvidersModule` be responsible for it's own separate context (`SoC`).
```ts
// ./providers/providers-routing.module.ts`:
{ path: '', component: ProviderRecordsComponent },
{ path: 'provider/new', component: ProviderNewComponent },
{ path: 'provider/:providerId', component: ProviderViewEditComponent },
```
Now let's take a look how with modules and lazy loading we can refactor our `AppRoutingModule`:
```ts
// app-routing.module.ts
const routes: Routes = [
    { path: '', redirectTo: '/learners', pathMatch: 'full' },
    { path: 'dashboard', loadChildren: () => import('./dashboard/dashboard.module').then(m => m.DashboardModule) },
    { path: 'reports', loadChildren: () => import('./reports/reports.module').then(m => m.ReportsModule)},
    { path: 'learners', loadChildren: () => import('./learners/learners.module').then(m => m.LearnersModule)},
    { path: 'providers', loadChildren: () => import('./providers/providers.module').then(m => m.ProvidersModule)},
    { path: 'system-events', loadChildren: () => import('./system/system-events.module').then(m => m.SystemEventsModule) },
    { path: 'system-mimic', loadChildren: () => import('./system-mimic/system-mimic.module').then(m => SystemMimicModule)),
    { path: 'users', loadChildren: () => import('./users/users.module').then(m => m.UsersModule)},
    { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)},
    { path: '**', component: LoggingInPleaseWaitComponent }
];
```
Another benefit of using lazy loading is that you can use guard called `CanLoad`, which can prevent users from loading module bundle when they don't have correct access or any other restrictions. For example: We can use `CanLoad` guard to prevent regular users from loading code responsible for `admin`, `system-mimic` or `system-events` modules.

Check for more: 
- https://angular.io/guide/lazy-loading-ngmodules
- https://angular.io/api/router/CanLoad
</details>

### Backend Architecture
1. Lack of unified architecture
2. Unstandardised solutions used without documentation.
3. Lack of standardised specification for routes. (REST, GraphQL, CRUD)

### ORM and Stored Procedures
1. SQL Project used instead of ORM - SQL Projects are under huge risk of being deprecated and many developers more famillar with Entity Framework and Code First appraoch which is used for years.
2. Most of developers this days don't develop SQL skills as it's legacy techology so it may generate problems with finding resources to maintain the project.
3. Relation Database is very uncommon approach for products like ILR. Where the data structure flexiblity is the key. Where writes and reads are equal or even reads are smaller.

## Dependencies
### Components Framework
### CSS Framework
![Maintainablity](https://img.shields.io/badge/Maintainablity-1.svg)
![Accessiblity](https://img.shields.io/badge/Accessiblity-gray.svg)
ILR has added Boostrap but most of the code don't actually use it, instead we have a mess with many custom classes. 

## Quality and Performance
### Forms
![Maintainablity](https://img.shields.io/badge/Maintainablity-1.svg)
![Reliability](https://img.shields.io/badge/Reliability-gray.svg)
![Accessiblity](https://img.shields.io/badge/Accessiblity-gray.svg)

Most of ILR project is about entering and validating user input. It's essential to have this part implemented in a possibly simplest way (`KISS`) to reduce cost of maintenance and amount of bugs related to it. Current implementation is completely oposite to that.

Problems:
- Loosing ablity of using core angular functionality such as `Reactive Forms` and `Template Driven Forms`.
- `Overengineering` - overcomplicated solutions used, many places even where custom solution is not required.
- `DRY` - the similar code in many places, triples amount of issues reported and work needs to be done to fix them.
- `Lava flow` - plenty unused code, left in a fear of breaking the system.
- Lack of custom solution documentation.

<details>
<summary>Show solution</summary>

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
</details>

### Input Binding Function Executions
![Performance](https://img.shields.io/badge/Performance-1.svg)

Using function invocation as an input in .HTML templates should be considered as a bad practice, when component change detection is set to default it will cause huge performance issues. This happens almost everywhere accross the project.

<details>
<summary>Show solution</summary>

For example, take a look at few first lines in `aim-completion-status.component.html` (random opened component, simplified):
```html
<data-mode-viewer [Mode]="getComponentDataMode()"></data-mode-viewer>
<t-lgt [Text]="getDetailsDates()"></t-lgt>
<t-lgt [Text]="getDetailsFullTitle()" ></t-lgt>
```

`[Model]` and both of the `[Text]` inputs use function result as a value. This will mean that this function needs to be called each time per change detection cycle. That results in houndreds of calls per minute as Angular trigger change detection even for events like `mousemove` or `wheel`.
This can be easy fixed by inlining function results or using getters which will have the same results:
```ts
// for example changing this:
public getComponentDataMode(): DataMode {
    return this.isComponentDataLoading() ? DataMode.Loading : DataMode.ViewAll;
}
// in to this
public get componentDataMode(): DataMode {
    return this.isLoading ? DataMode.Loading : DataMode.ViewAll;
}
```
then your HTML would look like that:
```html
<data-mode-viewer [Mode]="componentDataMode"></data-mode-viewer>
<t-lgt [Text]="detailsDates"></t-lgt>
<t-lgt [Text]="detailsFullTitle" ></t-lgt>
```

this will result in huge performance improvment. The problem can be also resolved by changing `ChangeDetectionStrategy` to `OnPush` and using observables for state changes.

Check for more:
https://angular.io/api/core/ChangeDetectionStrategy
</details> 


### State Managment

### Pixels vs Rems 
![Performance](https://img.shields.io/badge/Accessibility-1.svg)

A lot of pixes used in stylesheets. Pixels are not getting scaled when you change font-size in browser which may make layouts unresponsive. 

Solution: Replace all `px` units with `rems`.

### Overcomplicated HTML Templates
![Maintainablity](https://img.shields.io/badge/Maintainablity-grey.svg)
![Performance](https://img.shields.io/badge/Performance-grey.svg)

Some of HTML templates are overcomplicated. As we all know DOM tree is quite heavy and unoptimized templates are not a good friend in terms of application speed.

<details>
<summary>Show solution</summary>
Optimize HTML templates by moving all code responsible for styling to CSS and keeping them as clean as they can be.

```html
<!-- simplified learner-information.component.html -->
<fetching-data-spinner></fetching-data-spinner>

<div *ngIf="dataLoaded">
    <h1></h1>
    <div> <!-- unnused div -->
        <t-drk Text="Learner information"></t-drk>
        <div class="figma-standard-double-spacer"></div> <!-- spacing div element -->
        <t-drk Text="Fill in your learner’s personal information and learning details."></t-drk>
        <div class="figma-standard-quad-spacer"></div> <!-- spacing div element -->
        <div class="figma-standard-line-thin-full-lgt"></div> <!-- spacing div element -->
        <div *ngFor="let groupedField of getFilteredLearnerInformationGroups()"> <!-- replace by ng-container -->
            <div class="figma-standard-quad-spacer"></div> <!-- spacing div element -->
            <t-drk  *ngIf="groupedField.group === 1" Text="Personal information"></t-drk>
            <t-drk  *ngIf="groupedField.group === 2" Text="Current address"></t-drk>
            <t-drk  *ngIf="groupedField.group === 3" Text="Contact preferences"></t-drk>
            <t-drk  *ngIf="groupedField.group === 4" Text="Learning details"></t-drk>
            <div class="figma-standard-full-spacer"></div> <!-- spacing div element -->
            <div *ngFor="let field of groupedField.fieldDefinition"> <!-- replace by ng-container -->
                <div [ngSwitch]="field.typeId" *ngIf="groupedField.group != 3"> <!-- replace by ng-container -->
                    <div *ngSwitchCase="fieldType.Int"> <!-- can be applied to element directly -->
                        <field-definition-textbox></field-definition-textbox>
                    </div>
                    <div *ngSwitchCase="fieldType.Long"> <!-- can be applied to element directly -->
                        <field-definition-textbox></field-definition-textbox>
                    </div>
                    <div *ngSwitchCase="fieldType.Date"> <!-- can be applied to element directly -->
                        <field-definition-datepicker></field-definition-datepicker>
                    </div>
                    <div *ngSwitchCase="fieldType.String"> <!-- can be applied to element directly -->
                        <field-definition-textbox></field-definition-textbox>
                    </div>
                    <div *ngSwitchCase="fieldType.Dropdown"> <!-- can be applied to element directly -->
                        <field-definition-dropdown></field-definition-dropdown>
                    </div>
                </div>
            </div>
            <contact-preference></contact-preference>
        </div>
    </div>
</div>
```
and the simplified version:
```html
<!-- simplified -->
<fetching-data-spinner></fetching-data-spinner>
<ng-container *ngIf="dataLoaded">
    <h1></h1>
    <t-drk Text="Learner information"></t-drk>
    <t-drk Text="Fill in your learner’s personal information and learning details."></t-drk>
    <div *ngFor="let groupedField of getFilteredLearnerInformationGroups()">
        <t-drk  *ngIf="groupedField.group === 1" Text="Personal information"></t-drk>
        <t-drk  *ngIf="groupedField.group === 2" Text="Current address"></t-drk>
        <t-drk  *ngIf="groupedField.group === 3" Text="Contact preferences"></t-drk>
        <t-drk  *ngIf="groupedField.group === 4" Text="Learning details"></t-drk>
        <ng-container *ngFor="let field of groupedField.fieldDefinition">
            <ng-container [ngSwitch]="field.typeId" *ngIf="groupedField.group != 3">
                <field-definition-textbox *ngSwitchCase="fieldType.Int"></field-definition-textbox>
                <field-definition-textbox *ngSwitchCase="fieldType.Long"></field-definition-textbox>
                <field-definition-datepicker *ngSwitchCase="fieldType.Date"></field-definition-datepicker>
                <field-definition-textbox  *ngSwitchCase="fieldType.String"></field-definition-textbox>
                <field-definition-dropdown *ngSwitchCase="fieldType.Dropdown"></field-definition-dropdown>
            </ng-container>
        </ng-container>
        <contact-preference></contact-preference>
    </div>
</ng-container>
```
Using `<ng-container>` instead of `<div>` will keep the DOM tree to be very minimal after compilation.
By removing elements responsible for spacing and letting typograpy and container element handle that will help them to be very clean. It's maybe not a huge imporvement but by appling some simple rules we can reduce amount of code in project by `~40%` which is a lot of time saved for maintenance.
</details>

### Unorganised Styles
### Redundand Wrappers
### Developer tools mixed with production code
### Unused Dependency Injection
### Fake Unit Testing
### Lack of Unit Testing
### Lack of Logging

## Configuration
### Secrets

## Deployment
### Pipelines
### Docker

## Others
### Authentication

## Teminology
- SOLID - Single Responsiblity, Open Closed, Liskov Substitution, Interface Segregation, Dependency Inversion

- Rigidity - happens when software that is static and hard to change. Rigidity is often observed when a small change forces a complete rebuild feature. Small changes should be able to be built, tested, and deployed very quickly and independently of each other. Long build times are a symptom of high coupling.

- Fragility - applies to software that is non modular and tightly coupled. A change to one part of your system should never break another part that is completely unrelated. Even related functionality should be decoupled enough to extend functionality without affecting related components. There are many engineering patterns and practices that facilitate flexibility, extensibility, adaptability, along with many other qualities that inhibit fragility.

- Magic Numbers - 

- SoC - https://en.wikipedia.org/wiki/Separation_of_concerns

- Big Ball Of Mud - https://en.wikipedia.org/wiki/Big_ball_of_mud

- Accidental complexity -

- Cargo cult programming - 

- KISS -

- DRY -

- Boat anchor - 

## Worth reading
 1. Useful Tips - https://devlead.io/DevTips
 2. Clean Code
 3. Unit Testing
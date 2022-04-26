
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
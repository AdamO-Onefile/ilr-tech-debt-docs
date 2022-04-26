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
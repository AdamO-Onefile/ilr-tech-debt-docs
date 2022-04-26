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

## Frontend
### Angular Modules
![Maintainablity](https://img.shields.io/badge/Maintainablity-1.svg)
![Performance](https://img.shields.io/badge/Performance-grey.svg)

At the moment we don't use modules so much and as this is the one of Angular fundamentals I think we should take a look on the current situation and think about what can be done in that area. Consequences that come from not having properly structured application are very serious due to high risk of dealing with [Big Ball of Mud](#big-ball-of-mud) antipattern.

Problems:
- Tight coupoling, possible `Big Ball of Mud`.
- Loosing ablity of using angular features such as `CanLoad`, `Lazy loading` or `Preloading`. 
- Application bundle size - we need to load etrie app when user access to our application.
- Fragility - Resistance of refactoring due to high regression risk.

### Angular Routing
![Performance](https://img.shields.io/badge/Performance-1.svg)
![Maintainablity](https://img.shields.io/badge/Maintainablity-grey.svg)
![Security](https://img.shields.io/badge/Security-grey.svg)

Problems:
- Loosing ablity of using angular features such as `Resolvers`, `canActivateChild`, `shared templates` and many others listed in [Angular Modules](#angular-modules).
- Route tree is flat which means that child route will never know about parent existance, core conceptual problem.
- Top level module holds information about low level modules routing and configuration.




### Components Framework
![Maintainablity](https://img.shields.io/badge/Maintainablity-1.svg)

ILR has more than one component framework used. Actually it has 3.
- ngBootstrap
- Material
- OneFile Component Library (Rapid)

That makes project hard to maintain and creates confusing scenarios where developers not sure which component should be used to develop features. We may want to use many third party libraries but components should not duplicate.

Solution: Decide and refactor the code to stick to the one of this frameworks. I would suggest to stop using ngBootstrap as it's deprecated.

### CSS Framework
![Maintainablity](https://img.shields.io/badge/Maintainablity-1.svg)
![Accessiblity](https://img.shields.io/badge/Accessiblity-gray.svg)

ILR has a Bootstrap as dependency but most of the code don't actually use it, instead we have a mess with many custom classes. 

Solution: Reduce amount of custom SCSS classes used and sick to the bootstrap or use OneFile Component Library to handle spacings.

### Forms
![Maintainablity](https://img.shields.io/badge/Maintainablity-1.svg)
![Reliability](https://img.shields.io/badge/Reliability-gray.svg)
![Accessiblity](https://img.shields.io/badge/Accessiblity-gray.svg)

Most of ILR project is about entering and validating user input. It's essential to have this part implemented in a possibly simplest way (`KISS`) to reduce cost of maintenance and amount of bugs related to it. Current implementation is completely oposite to that.

Problems:
- Loosing ability of using core angular functionality such as `Reactive Forms` and `Template Driven Forms`.
- `Overengineering` - overcomplicated solutions used, many places even where custom solution is not required.
- `DRY` - the similar code in many places, triples amount of issues reported and work needs to be done to fix them.
- `Lava flow` - plenty unused code, left in a fear of breaking the system.
- Lack of custom solution documentation.



### Input Binding Function Executions
![Performance](https://img.shields.io/badge/Performance-1.svg)

Using function invocation as an input in .HTML templates should be considered as a bad practice, when component change detection is set to default it will cause huge performance issues. This happens almost everywhere across the project.

### Lack of State Management 
![Reliability](https://img.shields.io/badge/Reliability-1.svg)
![Maintainablity](https://img.shields.io/badge/Maintainablity-grey.svg)

Lack of unified way to manage the state. Each service is written differently. Users static or reactive data which often leads to unplated views or unpredictable system failures.

Solution:
- Use state management system eg. NGRX.
or 
- Decide how we handle common scenarios document and stick to those decisions.

### Pixels vs Rems 
![Performance](https://img.shields.io/badge/Accessibility-1.svg)

A lot of pixes used in stylesheets. Pixels are not getting scaled when you change font-size in browser which may make layouts unresponsive. 

Solution: Replace all `px` units with `rems`.

### Overcomplicated HTML Templates
![Maintainablity](https://img.shields.io/badge/Maintainablity-grey.svg)
![Performance](https://img.shields.io/badge/Performance-grey.svg)

Some of HTML templates are overcomplicated. As we all know DOM tree is quite heavy and optimized templates are not a good friend in terms of application speed.

Solution:
Optimize HTML templates by moving all code responsible for styling to CSS and keeping them as clean as they can be.

### Unorganized Styles
Messy stylesheets.

### Lack of Unit Testing
Lack of unit testing on frontend app. If we want to write good maintainable and reusable code we need to learn how to do it. No matter if it's the backend or frontend.

## Backend
### Complexity of the current implementation
![Maintainablity](https://img.shields.io/badge/Maintainablity-success.svg)

We have to many abstraction layers. (Services, Fetchers, Collectors, DataAccess, Controllers, DTO's, Entities and Database structures). ILR may look complex but that's just a illusion created by `accidental complexity`. We have only 2 aggregates. Learner and Provider. Nothing else.
The amount of layers often leads developer to skip or even worst not to skip bunch of layers just to get the data to be returned. This makes mess and leaves code impossible to test without setting up hundreds of mocks. 

Problems:
- To many abstraction layers.
- Where the dependencies are point downwards they are crossing everywhere, lack of proper SoC. There is no control over that by the code itself (`Encapsulation`).
- Lack of usage of the the domain entities (bussing models that contain all business logic operations).

### Custom implementations
![Maintainablity](https://img.shields.io/badge/Maintainablity-success.svg)
![Reliability](https://img.shields.io/badge/Reliability-grey.svg)
![Security](https://img.shields.io/badge/Security-grey.svg)
- Background Processors - current implementation leaves us with 0 ability for scaling ILR. As we process asynchronously in memory.
- Memory Cache - same as above custom and undocumented code.
- Configuration provider - is overcomplicated where .NET provides tools to handle out of the box. New developers may find difficulties to deal with this.
- SQL builder - untested and undocumented homemade ORM system. Huge risk, better not to use, and deprecate as soon as possible.
- Web API has no any common convention followed, like REST/GraphQL/CRUD.

### Observability and Monitoring
![Reliability](https://img.shields.io/badge/Reliability-grey.svg)
![Security](https://img.shields.io/badge/Security-grey.svg)

Limited observability on the system is a huge problem. At the moment only one place were we can look on to what goes on with the system is System Events, which is a feature of ILR which requires ILR Web App and Database to work to be usable.


Solution: 
- use Logger configured by following .NET Official documentation (use external logging provider).
- add middleware to handle unhandled exception and report it.
- add a health check/s.

### Secrets
![Security](https://img.shields.io/badge/Security-success.svg)
Secrets are exposed to everyone who can read repo.

Solution:
- Move secrets to azure key-vault and use environment variables to override them for deployment
or
- Use gitcrypt. 

### SQL Project and Stored Procedures
![Maintainablity](https://img.shields.io/badge/Maintainablity-grey.svg)
![Reliability](https://img.shields.io/badge/Reliability-grey.svg)

- SQL projects and latest versions of .NET don't like each other, huge risk of problems with automation, virtualization, scaling and risk of being deprecated. 
- Stored procedures that contain bussing logic are impossible to unit test. It's kind of a bit obsolete to even think about unit testing DATA STORAGE.

## Terminology
- SOLID - Single Responsibility, Open Closed, Liskov Substitution, Interface Segregation, Dependency Inversion

- Rigidity - happens when software that is static and hard to change. Rigidity is often observed when a small change forces a complete rebuild feature. Small changes should be able to be built, tested, and deployed very quickly and independently of each other. Long build times are a symptom of high coupling.

- Fragility - applies to software that is non modular and tightly coupled. A change to one part of your system should never break another part that is completely unrelated. Even related functionality should be decoupled enough to extend functionality without affecting related components. There are many engineering patterns and practices that facilitate flexibility, extensibility, adaptability, along with many other qualities that inhibit fragility.


- SoC - https://en.wikipedia.org/wiki/Separation_of_concerns

- Big Ball Of Mud - https://en.wikipedia.org/wiki/Big_ball_of_mud

- Accidental complexity - https://en.wikipedia.org/wiki/No_Silver_Bullet

- Cargo cult programming - https://en.wikipedia.org/wiki/Cargo_cult_programming

- KISS - https://en.wikipedia.org/wiki/KISS_principle

- DRY - https://en.wikipedia.org/wiki/Don%27t_repeat_yourself

## Worth reading
 1. Useful Tips - https://devlead.io/DevTips
 2. Clean Code
 3. Unit Testing
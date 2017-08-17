# ngx-permissions

Permission based access control for your angular applications

## Table of contents

- [Installation](#installation)
- [Consuming library](#consuming-library)
- [Managing Permissions](#managing-permissions)
- [Controlling access in views](#controlling-access-in-views)
- [Usage with Routes](#usage-with-routes)
- [Development](#development)
- [License](#license)

## Installation

To install this library, run:

```bash
$ npm install ngx-permissions --save
```

## Consuming library

Once you have published your library to npm, you can import your library in any Angular application by running:

```bash
$ npm install ngx-permissions  --save
```

and then from your Angular `AppModule`:

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';

// Import your library
import { NgxPermissionsModule } from 'ngx-permissions';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,

    // Specify your library as an import
     NgxPermissionsModule.forRoot()
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

SharedModule

If you use a SharedModule that you import in multiple other feature modules, you can export the NgxPermissionsModule to make sure you don't have to import it in every module.
```typescript
@NgModule({
    exports: [
        CommonModule,
        NgxPermissionsModule
    ]
})
export class SharedModule { }
```
> Note: Never call a forRoot static method in the SharedModule. You might end up with different instances of the service in your injector tree. But you can use forChild if necessary.

Once your library is imported, you can use its components, directives and pipes in your Angular application:

Import service to the main application and load permissions

```typescript
import { Component, OnInit } from '@angular/core';
import { PermissionsService } from 'ngx-permissions';
import { HttpClient } from '@angular/common/http';
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {

  title = 'app';

   constructor(private permissionsService: PermissionsService,
               private http: HttpClient) {}

  ngOnInit(): void {
    const perm = ["ADMIN", "EDITOR"];

    this.permissionsService.loadPermissions(perm);
    
     this.http.get('url').subscribe((permissions) => {
       //const perm = ["ADMIN", "EDITOR"]; example of permissions
       this.permissionsService.loadPermissions(permissions);
    })
  }
}
```
### Managing permissions


Overview
----------------------------

1. [Introduction](#introduction)
2. [Defining permissions](#defining-permissions)
  1. [Individual permissions](#individual-permissions)
  2. [Multiple permissions](#multiple-permissions)
3. [Removing permissions](#removing-permissions)
4. [Retrieving permissions](#retrieving-permissions)

Introduction
----------------------------

Let's start with little explanation **what** permission is. Permission is the most atomic **ability** that a user can have 
in your application. So you can think about permission as a smallest action that user can do inside your site. 

But can `user` or `anonymous` be a permission? Technically yes, but from business point of view you should treat them 
as Roles that are more complex objects that can store more complex logic. 

> :bulb: **Note**   
> It's a good convention to start permission with a verb and combine them with resource or object, so permissions like `readDocuments` or `listSongs` 
are meaningful and easy to understand for other programmes. Notice that they are named lowerCamelCase for easy differentiation form roles.
 
Defining permissions
----------------------------
So, how do you tell Permission what does 'readDocuments' or 'listSongs' mean and how to know if the current user belongs
to those definitions?

Well, Permission allows you to set different 'permissions' definitions along with the logic that determines if the current 
session belongs to them. To do that library exposes special container `PermissionsService` that allows you to manipulate them freely.

### Individual permissions

To add permissions individually `PermissionsService` exposes method `addPermission` that generic usage is shown below or add as array: 

```typescript
[...]
 ngOnInit() {
    this.permissionsService.addPermission('changeSomething')
    this.permissionsService.addPermission(['changeSomething', 'anotherAlso'])
 }

```

### Multiple permissions

To define multiple permissions  method `loadPermissions` can be used. The only 
difference from `definePermission` is that it accepts `Array` of permission names instead of single one. 


Often meet example of usage is set of permissions (e.g. received from server after user login) that you will iterate over to 
check if permission is valid.

```typescript
const permissions = ['listMeeting', 'seeMeeting', 'editMeeting', 'deleteMeeting']
PermissionsService.loadPermissions(permissions) 
```
NOTE: This method will remove older permissions and pass only new;

Removing permissions
----------------------------

You can easily remove **all** permissions form the `PermissionsService` (e.g. after user logged out or switched profile) by calling:  

```typescript
PermissionsService.flushPermissions();
```

Alternatively you can use `removePermission` to delete defined permissions manually:

```typescript
PermissionsService.removePermission('user');
```

Retrieving permissions
----------------------------

And to get all user permissions use method `getPermissions` or use Observable `permissions$`:

```typescript
var permissions = PermissionsService.getPermissions();

PermissionsService.permissions$.subscribe((permissions) => {
    console.log(permissions)
})
```

## Controlling access in views

Overview
----------------------------

1. [Permission directive](#permission-directive)
  1. [Basic usage](#basic-usage)

Permission directive
----------------------------
  
Permission module exposes directive `permissions` that can show/hide elements of your application based on set of permissions.

Permission directive accepts several attributes:

| Attribute             | Value                    | Description      |
| :----------------------|:------------------------:| :----------------|
| `permissionsOnly`     | <code>[String &#124; String[]]</code>   | Single or multiple permissions allowed to access content | 
| `permissionsExcept`   | <code>[String &#124; String[]]</code>   | Single or multiple permissions denied to access content|

### Basic usage

Directives accepts either single permission that has to be met in order to display it's content:
 
```html
<ng-template permissions [permissionsOnly]="['ADMIN']">
    <div>You can see this text congrats</div>
 </ng-template>
 
 <ng-template permissions [permissionsExcept]="['JOHNY']">
   <div> All will see it except JOHNY</div>
 </ng-template>
```

Or set of permissions separated by 'coma':

```html
<ng-template permissions [permissionsOnly]="['ADMIN', 'GUEST']">
    <div>You can see this text congrats</div>
</ng-template>

 <ng-template permissions [permissionsExcept]="['ADMIN', 'JOHNY']">
   <div>All will see it except admin and Johny</div>
 </ng-template>
```
 
Usage with Routes
----------------------------

1. [Introduction](#introduction)
2. [Property only and except](#property-only-and-except)
  1. [Single permission/role](#single-permissionrole)
  2. [Multiple permissions/roles](#multiple-permissionsroles) 
3. [Property redirectTo](#property-redirectto)
  1. [Single rule redirection](#single-redirection-rule)

Introduction
----------------------------

Now you are ready to start working with controlling access to the states of your application. In order to restrict any state ngx-permission rely on angular-route's `data` property, reserving key `permissions` allowing to define authorization configuration.

Permissions object accepts following properties:

| Property        | Accepted value                   |
| :-------------- | :------------------------------- |
| `only`          | [`String`\|`Array`]              |
| `except`        | [`String`\|`Array`]              |
| `redirectTo`    | [`String`]                       |

Property only and except
----------------------------

Property `only`:
  - is used to explicitly define permission or role that are allowed to access the state   
  - when used as `String` contains single permission or role
  - when used as `Array` contains set of permissions and/or roles

Property `except`: 
  - is used to explicitly define permission or role that are denied to access the state
  - when used as `String` contains single permission or role
  - when used as `Array` contains set of permissions and/or roles

[//]: <> (> :fire: **Important**   
          > If you combine both `only` and `except` properties you have to make sure they are not excluding each other, because denied roles/permissions would not allow access the state for users even if allowed ones would pass them.   
)
 
#### Single permission/role 

In simplest cases you allow users having single role permission to access the state. To achieve that you can pass as `String` desired role/permission to only/except property:

```typescript
import { RouterModule, Routes } from '@angular/router';
import { NgModule } from '@angular/core';
import { HomeComponent } from './home/home.component';
import { PermissionsGuard } from 'ngx-permissions';

const appRoutes: Routes = [
  { path: 'home',
    component: HomeComponent,
    canActivate: [PermissionsGuard],
    data: {
      permissions: {
        only: 'ADMIN'
      }
    }
  },
];
@NgModule({
  imports: [
    RouterModule.forRoot(appRoutes)
  ],
  exports: [
    RouterModule
  ]
})
export class AppRoutingModule {}

```

In given case when user is trying to access `home` state `PermissionsGuard` service is called checking if `isAuthorized` permission is valid: 
  - if permission definition is not found it stops transition

#### Multiple permissions/roles 

Often several permissions/roles are sufficient to allow/deny user to access the state. Then array value comes in handy:  

```typescript
import { RouterModule, Routes } from '@angular/router';
import { NgModule } from '@angular/core';
import { HomeComponent } from './home/home.component';
import { PermissionsGuard } from 'ngx-permissions';

const appRoutes: Routes = [
  { path: 'home',
    component: HomeComponent,
    canActivate: [PermissionsGuard],
    data: {
      permissions: {
        only: ['ADMIN', 'MODERATOR']
      }
    }
  },
];
@NgModule({
  imports: [
    RouterModule.forRoot(appRoutes)
  ],
  exports: [
    RouterModule
  ]
})
export class AppRoutingModule {}
```

When `PermissionGuard` service will be called it would expect user to have either `ADMIN` or `MODERATOR` permissions to pass him to `home` route.

[//]: <> (> :bulb: **Note**   
          > Between values in array operator **OR** is used to create alternative. If you need **AND** operator between permissions define additional `PermRole` containing set of those. 
)

Property redirectTo
----------------------------

Property redirectTo:
  - when used as `String` defines single redirection rule

### Single redirection rule

In case you want to redirect to some specific state when user is not authorized pass to `redirectTo` path of that route.

```typescript
import { RouterModule, Routes } from '@angular/router';
import { NgModule } from '@angular/core';
import { HomeComponent } from './home/home.component';
import { PermissionsGuard } from 'ngx-permissions';

const appRoutes: Routes = [
  { path: 'home',
    component: HomeComponent,
    canActivate: [PermissionsGuard],
    data: {
      permissions: {
        only: ['ADMIN', 'MODERATOR'],
        redirectTo: 'another-route'
      }
    }
  },
];
@NgModule({
  imports: [
    RouterModule.forRoot(appRoutes)
  ],
  exports: [
    RouterModule
  ]
})
export class AppRoutingModule {}
```

----------------------------

| --- |
## Development

To generate all `*.js`, `*.d.ts` and `*.metadata.json` files:

```bash
$ npm run build
```

To lint all `*.ts` files:

```bash
$ npm run lint
```

## License

MIT © [Oleksandr Khymenko](mailto:alexanderKhymenko@gmail.com)

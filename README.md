# TypeScript Style Guide Draft

This is the AngularJS coding style guide for using TypeScript which is used internally at PernixData.

## General Rules
  - For TypeScript style guide, please refer to [this](https://github.com/Platypi/style_typescript).
  - Controllers, services, directives are written in format of TypeScript class.
  - Do not use unnecessary `public` modifier which is default.
  - Use single TypeScript module for entire project in order to avoid `require` syntax and use angularJS module system

### Dependency Injection
  - Define dependencies in the `static` property `$inject`.
  - Include dependencies as class constructor parameters using the same names in the same order.
  - Store dependencies as class properties using name without `$` if possible.
  - 
  
  ```typescript
  // bad order 
  class AppCtrl {
      static $inject: string[] = ['myService', '$scope'];
      
      constructor($scope: angular.IScope
                  myService: MyService) {
          ...
      }
  }
  
  // bad name
  class AppCtrl {
      static $inject: string[] = ['$scope', 'myService'];
      
      constructor(s: angular.IScope,
                  srv: MyService) {
          ...
      }
  }
  
  // good
  class AppCtrl {
      static $inject: string[] = ['$scope', 'myService'];
      
      // dependencies
      scope: angular.IScope;
      myService: MyService;
      
      constructor($scope: angular.IScope,
                  myService: MyService) {
          this.scope = $scope;
          this.myService = myService;
      }
  }
  ```
  
  
## Controller
  - Use UpperCamelCase with `Ctrl` suffix for naming controller class
  - Avoid using `$scope` as much as possible.
  - Define properies and functions as properties of controller class rather than `$scope` properties.
  - Create a custom scope interface extending angular scope interface if adding properties to `$scope` is unavoiable.
  - Controller is bound to view in angular ui router state definition.
  - Use controller alias in order to access properties and functions defined as controller properties
  - Use lowerCamelCase with the same name as controller class for controller alias
  - Define necessary interfaces or classes within the same file and do not export them if these types are only used for this controller
  - Define functions as `private` when they are not supposed to used outside the controller class 
  - 
  
  ```typescript
  // custom scope interface
  interface IMyScope extends angular.IScope {
      unavoiableProp: string;
      ...
  }
  
  // controller definition
  class AppCtrl {
      static $inject: string[] = ['$scope', 'myService'];
      
      // dependencies
      scope: IMyScope;
      myService: MyService;
      
      // properties
      name: string;
      
      constructor($scope: IMyScope,
                  myService: MyService) {
          this.scope = $scope;
          this.myService = myService;
          
          this.name = 'App controller';
      }
      
      simpleFn(n: number): number {
          return n + n;
      }
  }
  
  // create controller with alias in ui router state provider
  $stateProvider
      .state('route', {
          controller: AppCtrl,
          controllerAs: 'appCtrl',
          ...
      })
  ```
  
  ```html
  <!-- Note this part of HTML should be within the ui-view of this controller's parent view -->
  <div>
      <span>Controller Name</span>
      <span ng-bind='appCtrl.name'></span>
  </div>
  
  <div>
      <span>Click Me</span>
      <span ng-click='appCtrl.simpleFn(1)'></span>
  </div>
  ```
  
## Service
  - Use UpperCamelCase with `Service` suffix for naming service class.
  - Use lowerCamelCase for naming angular service.
  - Do necessary service initialization in `constructor`.
  - All public functions must have a comment block `/**...*/` using [JSDoc](http://usejsdoc.org/) style comments. 
  - Define functions as `private` when they are not supposed to used outside the service class.
  - Register service using `module.service('angularServiceName', ServiceClass)`.
  - 

  ```typescript
  // service definition
  class MyService {
      static $inject: string[] = ['$scope'];
      
      // dependencies
      scope: IMyScope;
      
      // properties
      name: string;
      
      constructor($scope: IMyScope) {
          this.scope = $scope;
          
          this.name = 'My Service';
      }
      
      doStuff(...): Type {
          ...
          this.doStuffPrivate();
          ...
      }
      
      private doStuffPrivate(): void {
          ...
      }
  }
  
  // register as angular service
  angular.module('moduleName', [...]).service('myService', MyService);
```


# TypeScript Style Guide

This is the AngularJS coding guide for using TypeScript which is used internally at PernixData.

## General Rules
  - For TypeScript style guide, please refer to [this](https://github.com/Platypi/style_typescript).
  - Controllers, services, directives are written in format of TypeScript class.
  - Do not use unnecessary `public` modifier which is default.
  - Use `private` for private class properties, otherwise they are `public` by default. 
  - Use single TypeScript module for entire project in order to avoid `require` syntax and use angularJS module system
  - 

### Dependency Injection
  - Define dependencies in the `static` property `$inject`.
  - Include dependencies as class constructor parameters using the same names in the same order.
  - Store dependencies as class properties using name without `$` if possible.
  - Don't use `$injector` service when direct dependency injection is possible (e.g. use dependency injection for controllers, services, filters, directives, and configs.
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
  
  - Use `$injector` for base angular classes (controller class, service class, config class, directive class and filter class) when OO design is used and direct dependency injection is not available.
  - `$scope` is not available from `$injector`

  ```typescript
  export class BaseCtrl {
      service: IService;
      scope: angular.IScope;
  
      constructor($injector: ng.auto.IInjectorService, scope: angular.IScope, ...) {
          this.service = <IService> $injector.get('serviceName');
          this.scope = scope;
          ...
      }
  }
  ```
  
  - Directly pass dependencies needed to regular class when dependency injection is not possible
  - 
  ```typescript
  class MyClass {
      service: IService
      
      constructor(service: IService, ...) {
          this.service = service;
          ...
      }
  }
  
  export class MyCtrl {
      $inject: string[] = ['service', ...];
      
      constructor(service: IService, ...) {
          // direct pass dependencies here
          let myClass = new MyClass(service);
      }
  }
  ```
  
### CallBack Function
  - Use arrow function for callback function to resolve 'this' during compile time
  - When callback function is provided via a class function, use [automatic class binding](http://stackoverflow.com/questions/20627138/typescript-this-scoping-issue-when-called-in-jquery-callback) technique
  - 
  
  ```typescript
  class MyService {
      static $inject: string[] = [''];
      
      constructor() {
      }
      
      callBackFn: type = (...) => {
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
      
      /**
       * function description goes here.
       */
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

## Filter
  - Use UpperCamelCase with `Filter` suffix for naming service class.
  - Use the same UpperCamelCase for naming angular filter.
  - Do necessary service initialization in `constructor`.
  - Register service using `module.filter('MyFilter', MyFilter)`.
  - Add filter type to a unified custom filter interface. Otherwise, `$filter('FilterName')` doesn't know the type info (see example below)
  - 
  ```typescript
  // add filter type to the unified interface
  export interface IMyFilterService extends angular.IFilterService {
      (name: 'MyFilter'): (value: string | number) => string;
  }

  export class MyFilter {
      static $inject: string[] = [];
      
      constructor() {
          return (value: string | number): string => {
              ...
              return something;
          }
      }
  }

  // register as angular filter  
  angular.module('module-name', []).filter('MyFilter', MyFilter);
  ```


## Directive
  - Use UpperCamelCase with `Diretive` suffix for naming service class.
  - Use the same lowerCamelCase without `Diretive` for naming angular diretive.
  - The directive class should implements angular.IDirective
  - Define angular directive options (e.g. restrict and templateUrl) as class properties of type string
  - Define dependencies in the constructor parameters
  - Define `link` as arrow function
  - Create a `static` method called factory which is used to register directive
  - `factory` function returns a new instance of directive class (see example below)
  - Register diretive using `module.directive('myService', MyDirective.factory());`
  
  ```typescript
  class MyDirective implements angular.IDirective {
      myService: IMyService;
  
      restrict: string = 'E';
      templateUrl: string = 'url/mydirective.html';

      constructor(myService: IMyService) {
          this.myService = myService;
      }

      link = (scope: IMyScope, element: angular.IAugmentedJQuery, attrs: IMyAttrs) => {
          ...
      };

      static factory(): ng.IDirectiveFactory {
          const directive: ng.IDirectiveFactory = (myService: IMyService) => new MyDirective(myService);
          directive.$inject = ['myService'];
          return directive;
      }
  }

  angular.module('module-name', []).directive('myService', MyDirective.factory());
  ```

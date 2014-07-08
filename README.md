REQUIREJS API
=====================

##### Table of Contents

1. [사용법](#Usage)  
 1. [자바스크립트 파일 로딩](#Load_JavaScript_Files)  
 1. [data-main Entry Point](#data-main_Entry_Point)  
 1. [Define a Module](#Define_a_Module)  
  1. [Simple Name/Value Pairs](#Simple_Name_Value_Pairs)  
  1. [Definition Functions](#Definition_Functions)  
  1. [Definition Functions with Dependencies](#Definition_Functions_with_Dependencies)  
  1. [Define a Module as a Function](#Define_a_Module_as_a_Function)  
  1. [Define a Module with Simplified CommonJS Wrapper](#Define_a_Module_with_Simplified_CommonJS_Wrapper)  
  1. [Define a Module with a name](#)  
  1. [Other Module Notes](#)  
  1. [Circular Dependencies](#)  
  1. [Specify a JSONP Service Dependency](#)  
  1. [Undefining a Module](#)  
1. [Mechanics](#)  
1. [Configuration Options](#)  
1. [Advanced Usage](#)  
 1. [Loading Modules from Packages](#)  
 1. [Multiversion Support](#)  
 1. [Loading Code After Page Load](#)  
 1. [Web Worker Support](#)  
 1. [Rhino Support](#)  
 1. [Handling Errors](#)  
1. [Loader Plugins](#)  
 1. [Specify a Text File Dependency](#)  
 1. [Page Load Event Support/DOM Ready](#)  
 1. [Define an I18N Bundle](#)  

<a name="Usage">
# 사용법

<a name="Load_JavaScript_Files">
## 자바스크립트 파일 로딩

RequireJS takes a different approach to script loading than traditional &lt;script&gt; tags. While it can also run fast and optimize well, the primary goal is to encourage modular code. As part of that, it encourages using module IDs instead of URLs for script tags.

RequireJS는 전형적인 &lt;script&gt; 태그보다 script를 읽는 또 다른 접근법이다. 빠르고 최적화되게 실행할 수 있자만, 기본 목표는 모듈화된 코드를 만드는 것이다. 그 일환으로, script 태그를 위해 URL 대신에 모듈 ID를 사용하는 것을 권장한다.

RequireJS loads all code relative to a baseUrl. The baseUrl is normally set to the same directory as the script used in a data-main attribute for the top level script to load for a page. The data-main attribute is a special attribute that require.js will check to start script loading. This example will end up with a baseUrl of scripts:

```html
<!--This sets the baseUrl to the "scripts" directory, and
    loads a script that will have a module ID of 'main'-->
<script data-main="scripts/main.js" src="scripts/require.js"></script>
```

Or, baseUrl can be set manually via the RequireJS config. If there is no explicit config and data-main is not used, then the default baseUrl is the directory that contains the HTML page running RequireJS.

RequireJS also assumes by default that all dependencies are scripts, so it does not expect to see a trailing ".js" suffix on module IDs. RequireJS will automatically add it when translating the module ID to a path. With the paths config, you can set up locations of a group of scripts. All of these capabilities allow you to use smaller strings for scripts as compared to traditional &lt;script&gt; tags.

There may be times when you do want to reference a script directly and not conform to the "baseUrl + paths" rules for finding it. If a module ID has one of the following characteristics, the ID will not be passed through the "baseUrl + paths" configuration, and just be treated like a regular URL that is relative to the document:

* Ends in ".js".
* Starts with a "/".
* Contains an URL protocol, like "http:" or "https:".

In general though, it is best to use the baseUrl and "paths" config to set paths for module IDs. By doing so, it gives you more flexibility in renaming and configuring the paths to different locations for optimization builds.

Similarly, to avoid a bunch of configuration, it is best to avoid deep folder hierarchies for scripts, and instead either keep all the scripts in baseUrl, or if you want to separate your library/vendor-supplied code from your app code, use a directory layout like this:
* www/
 * index.html
 * js/
  * app/
   * sub.js
  * lib/
   * jquery.js
   * canvas.js
  * app.js

in index.html:
````html
<script data-main="js/app.js" src="js/require.js"></script>
````
and in app.js:
````javascript
requirejs.config({
    //By default load any module IDs from js/lib
    baseUrl: 'js/lib',
    //except, if the module ID starts with "app",
    //load it from the js/app directory. paths
    //config is relative to the baseUrl, and
    //never includes a ".js" extension since
    //the paths config could be for a directory.
    paths: {
        app: '../app'
    }
});

// Start the main app logic.
requirejs(['jquery', 'canvas', 'app/sub'],
function   ($,        canvas,   sub) {
    //jQuery, canvas and the app/sub module are all
    //loaded and can be used here now.
});
````
Notice as part of that example, vendor libraries like jQuery did not have their version numbers in their file names. It is recommended to store that version info in a separate text file if you want to track it, or if you use a tool like volo, it will stamp the package.json with the version information but keep the file on disk as "jquery.js". This allows you to have the very minimal configuration instead of having to put an entry in the "paths" config for each library. For instance, configure "jquery" to be "jquery-1.7.2".

Ideally the scripts you load will be modules that are defined by calling define(). However, you may need to use some traditional/legacy "browser globals" scripts that do not express their dependencies via define(). For those, you can use the shim config. To properly express their dependencies.

If you do not express the dependencies, you will likely get loading errors since RequireJS loads scripts asynchronously and out of order for speed.

<a name="data-main_Entry_Point">
## data-main Entry Point

The data-main attribute is a special attribute that require.js will check to start script loading:
````html
<!--when require.js loads it will inject another script tag
    (with async attribute) for scripts/main.js-->
<script data-main="scripts/main" src="scripts/require.js"></script>
````
You will typically use a data-main script to set configuration options and then load the first application module. Note: the script tag require.js generates for your data-main module includes the async attribute. This means that you cannot assume that the load and execution of your data-main script will finish prior to other scripts referenced later in the same page.

For example, this arrangement will fail randomly when the require.config path for the 'foo' module has not been set prior to it being require()'d later:
````html
<script data-main="scripts/main" src="scripts/require.js"></script>
<script src="scripts/other.js"></script>
````
````javascript
// contents of main.js:
require.config({
    paths: {
        foo: 'libs/foo-1.1.3'
    }
});
````
````javascript
// contents of other.js:

// This code might be called before the require.config() in main.js
// has executed. When that happens, require.js will attempt to
// load 'scripts/foo.js' instead of 'scripts/libs/foo-1.1.3.js'
require( ['foo'], function( foo ) {

});
````

<a name="Define_a_Module">
## Define a Module

A module is different from a traditional script file in that it defines a well-scoped object that avoids polluting the global namespace. It can explicitly list its dependencies and get a handle on those dependencies without needing to refer to global objects, but instead receive the dependencies as arguments to the function that defines the module. Modules in RequireJS are an extension of the Module Pattern, with the benefit of not needing globals to refer to other modules.

The RequireJS syntax for modules allows them to be loaded as fast as possible, even out of order, but evaluated in the correct dependency order, and since global variables are not created, it makes it possible to load multiple versions of a module in a page.

(If you are familiar with or are using CommonJS modules, then please also see CommonJS Notes for information on how the RequireJS module format maps to CommonJS modules).

There should only be one module definition per file on disk. The modules can be grouped into optimized bundles by the optimization tool.

<a name="Simple_Name_Value_Pairs">
### Simple Name/Value Pairs

If the module does not have any dependencies, and it is just a collection of name/value pairs, then just pass an object literal to define():
````javascript
//Inside file my/shirt.js:
define({
    color: "black",
    size: "unisize"
});
````

<a name="Definition_Functions">
### Definition Functions

If the module does not have dependencies, but needs to use a function to do some setup work, then define itself, pass a function to define():
````javascript
//my/shirt.js now does setup work
//before returning its module definition.
define(function () {
    //Do setup work here

    return {
        color: "black",
        size: "unisize"
    }
});
````
<a name="Definition_Functions_with_Dependencies">
### Definition Functions with Dependencies
If the module has dependencies, the first argument should be an array of dependency names, and the second argument should be a definition function. The function will be called to define the module once all dependencies have loaded. The function should return an object that defines the module. The dependencies will be passed to the definition function as function arguments, listed in the same order as the order in the dependency array:
````javascript
//my/shirt.js now has some dependencies, a cart and inventory
//module in the same directory as shirt.js
define(["./cart", "./inventory"], function(cart, inventory) {
        //return an object to define the "my/shirt" module.
        return {
            color: "blue",
            size: "large",
            addToCart: function() {
                inventory.decrement(this);
                cart.add(this);
            }
        }
    }
);
````
In this example, a my/shirt module is created. It depends on my/cart and my/inventory. On disk, the files are structured like this:
* my/cart.js
* my/inventory.js
* my/shirt.js

The function call above specifies two arguments, "cart" and "inventory". These are the modules represented by the "./cart" and "./inventory" module names.

The function is not called until the my/cart and my/inventory modules have been loaded, and the function receives the modules as the "cart" and "inventory" arguments.

Modules that define globals are explicitly discouraged, so that multiple versions of a module can exist in a page at a time (see Advanced Usage). Also, the order of the function arguments should match the order of the dependencies.

The return object from the function call defines the "my/shirt" module. By defining modules in this way, "my/shirt" does not exist as a global object.

<a name="Define_a_Module_as_a_Function">
### Define a Module as a Function
Modules do not have to return objects. Any valid return value from a function is allowed. Here is a module that returns a function as its module definition:
````javascript
//A module definition inside foo/title.js. It uses
//my/cart and my/inventory modules from before,
//but since foo/title.js is in a different directory than
//the "my" modules, it uses the "my" in the module dependency
//name to find them. The "my" part of the name can be mapped
//to any directory, but by default, it is assumed to be a
//sibling to the "foo" directory.
define(["my/cart", "my/inventory"],
    function(cart, inventory) {
        //return a function to define "foo/title".
        //It gets or sets the window title.
        return function(title) {
            return title ? (window.title = title) :
                   inventory.storeName + ' ' + cart.name;
        }
    }
);
````
<a name="Define_a_Module_with_Simplified_CommonJS_Wrapper">
#### Define a Module with Simplified CommonJS Wrapper
If you wish to reuse some code that was written in the traditional CommonJS module format it may be difficult to re-work to the array of dependencies used above, and you may prefer to have direct alignment of dependency name to the local variable used for that dependency. You can use the simplified CommonJS wrapper for those cases:
````javascript
define(function(require, exports, module) {
	var a = require('a'),
		b = require('b');

	//Return the module value
	return function () {};
}
);
````
This wrapper relies on Function.prototype.toString() to give a useful string value of the function contents. This does not work on some devices like the PS3 and some older Opera mobile browsers. Use the optimizer to pull out the dependencies in the array format for use on those devices.

More information is available on the CommonJS page, and in the in the "Sugar" section in the Why AMD page.

<a name="Define_a_Module_with_a_Name">
### Define a Module with a Name
You may encounter some define() calls that include a name for the module as the first argument to define():
````javascript
//Explicitly defines the "foo/title" module:
define("foo/title",
	["my/cart", "my/inventory"],
	function(cart, inventory) {
		//Define foo/title object in here.
	}
);
````
These are normally generated by the optimization tool. You can explicitly name modules yourself, but it makes the modules less portable -- if you move the file to another directory you will need to change the name. It is normally best to avoid coding in a name for the module and just let the optimization tool burn in the module names. The optimization tool needs to add the names so that more than one module can be bundled in a file, to allow for faster loading in the browser.

<a name="Other_Module_Notes">
### Other Module Notes
One module per file.: Only one module should be defined per JavaScript file, given the nature of the module name-to-file-path lookup algorithm. Multiple modules will be grouped into optimized files by the optimization tool, but you should only use the optimization tool to place more than one module in a file.

Relative module names inside define(): For require("./relative/name") calls that can happen inside a define() function call, be sure to ask for "require" as a dependency, so that the relative name is resolved correctly:
````javascript
define(["require", "./relative/name"], function(require) {
	var mod = require("./relative/name");
});
````
Or better yet, use the shortened syntax that is available for use with translating CommonJS modules:
````javascript
define(function(require) {
    var mod = require("./relative/name");
});
````
This form will use Function.prototype.toString() to find the require() calls, and add them to the dependency array, along with "require", so the code will work correctly with relative paths.

Relative paths are really useful if you are creating a few modules inside a directory, so that you can share the directory with other people or other projects, and you want to be able to get a handle on the sibling modules in that directory without having to know the directory's name.

Generate URLs relative to module: You may need to generate an URL that is relative to a module. To do so, ask for "require" as a dependency and then use require.toUrl() to generate the URL:
````javascript
define(["require"], function(require) {
    var cssUrl = require.toUrl("./style.css");
});
````
Console debugging: If you need to work with a module you already loaded via a require(["module/name"], function(){}) call in the JavaScript console, then you can use the require() form that just uses the string name of the module to fetch it:
````javascript
require("module/name").callSomeFunction()
````
Note this only works if "module/name" was previously loaded via the async version of require: require(["module/name"]). If using a relative path, like './module/name', those only work inside define

<a name="Circular_Dependencies">
### Circular Dependencies
If you define a circular dependency ("a" needs "b" and "b" needs "a"), then in this case when "b"'s module function is called, it will get an undefined value for "a". "b" can fetch "a" later after modules have been defined by using the require() method (be sure to specify require as a dependency so the right context is used to look up "a"):
````javascript
//Inside b.js:
define(["require", "a"],
    function(require, a) {
        //"a" in this case will be null if "a" also asked for "b",
        //a circular dependency.
        return function(title) {
            return require("a").doSomething();
        }
    }
);
````
Normally you should not need to use require() to fetch a module, but instead rely on the module being passed in to the function as an argument. Circular dependencies are rare, and usually a sign that you might want to rethink the design. However, sometimes they are needed, and in that case, use require() as specified above.

If you are familiar with CommonJS modules, you could instead use exports to create an empty object for the module that is available immediately for reference by other modules. By doing this on both sides of a circular dependency, you can then safely hold on to the the other module. This only works if each module is exporting an object for the module value, not a function:
````javascript
//Inside b.js:
define(function(require, exports, module) {
    //If "a" has used exports, then we have a real
    //object reference here. However, we cannot use
    //any of "a"'s properties until after "b" returns a value.
    var a = require("a");

    exports.foo = function () {
        return a.bar();
    };
});
````
Or, if you are using the dependency array approach, ask for the special 'exports' dependency:
````javascript
//Inside b.js:
define(['a', 'exports'], function(a, exports) {
    //If "a" has used exports, then we have a real
    //object reference here. However, we cannot use
    //any of "a"'s properties until after "b" returns a value.

    exports.foo = function () {
        return a.bar();
    };
});
````

<a name="Specify_a_JSONP_Service_Dependency">
### Specify a JSONP Service Dependency
JSONP is a way of calling some services in JavaScript. It works across domains and it is an established approach to calling services that just require an HTTP GET via a script tag.

To use a JSONP service in RequireJS, specify "define" as the callback parameter's value. This means you can get the value of a JSONP URL as if it was a module definition.

Here is an example that calls a JSONP API endpoint. In this example, the JSONP callback parameter is called "callback", so "callback=define" tells the API to wrap the JSON response in a "define()" wrapper:
````javascript
require(["http://example.com/api/data.json?callback=define"],
    function (data) {
        //The data object will be the API response for the
        //JSONP data call.
        console.log(data);
    }
);
````
This use of JSONP should be limited to JSONP services for initial application setup. If the JSONP service times out, it means other modules you define via define() may not get executed, so the error handling is not robust.

Only JSONP return values that are JSON objects are supported. A JSONP response that is an array, a string or a number will not work.

This functionality should not be used for long-polling JSONP connections -- APIs that deal with real time streaming. Those kinds of APIs should do more script cleanup after receiving each response, and RequireJS will only fetch a JSONP URL once -- subsequent uses of the same URL as a dependency in a require() or define() call will get a cached value.

Errors in loading a JSONP service are normally surfaced via timeouts for the service, since script tag loading does not give much detail into network problems. To detect errors, you can override requirejs.onError() to get errors. There is more information in the Handling Errors section.

<a name="Undefining_a_Module">
### Undefining a Module
There is a global function, requirejs.undef(), that allows undefining a module. It will reset the loader's internal state to forget about the previous definition of the module.

However, it will not remove the module from other modules that are already defined and got a handle on that module as a dependency when they executed. So it is really only useful to use in error situations when no other modules have gotten a handle on a module value, or as part of any future module loading that may use that module. See the errback section for an example.

If you want to do more sophisticated dependency graph analysis for undefining work, the semi-private onResourceLoad API may be helpful.

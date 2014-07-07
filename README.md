REQUIREJS API
=====================

##### Table of Contents

1. [사용법](#Usage)  
 1. [자바스크립트 파일 로딩](#Load_JavaScript_Files)  
 1. [data-main Entry Point](#data-main Entry Point)  
 1. [Define a Module](#)  
  1. [Simple Name/Value Pairs](#)  
  1. [Definition Functions](#)  
  1. [Definition Functions with Dependencies](#)  
  1. [Define a Module as a Function](#)  
  1. [Define a Module with Simplified CommonJS Wrapper](#)  
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

<a name="data-main Entry Point">
## data-main Entry Point

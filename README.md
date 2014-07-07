REQUIREJS API
=====================

##### Table of Contents
 [Usage](#Usage)
 [Load JavaScript Files](##Load JavaScript Files)
 [data-main Entry Point](##data-main Entry Point)
 [Define a Module](#)
 [Simple Name/Value Pairs](#)
 [Definition Functions](#)
 [Definition Functions with Dependencies](#)
 [Define a Module as a Function](#)
 [Define a Module with Simplified CommonJS Wrapper](#)
 [Define a Module with a name](#)
 [Other Module Notes](#)
 [Circular Dependencies](#)
 [Specify a JSONP Service Dependency](#)
 [Undefining a Module](#)
 [Mechanics](#)
 [Configuration Options](#)
 [Advanced Usage](#)
 [Loading Modules from Packages](#)
 [Multiversion Support](#)
 [Loading Code After Page Load](#)
 [Web Worker Support](#)
 [Rhino Support](#)
 [Handling Errors](#)
 [Loader Plugins](#)
 [Specify a Text File Dependency](#)
 [Page Load Event Support/DOM Ready](#)
 [Define an I18N Bundle](#)


# Usage

## Load JavaScript Files

RequireJS takes a different approach to script loading than traditional &lt;script&gt; tags. While it can also run fast and optimize well, the primary goal is to encourage modular code. As part of that, it encourages using module IDs instead of URLs for script tags.

RequireJS는 전형적인 &lt;script&gt; 태그보다 script를 읽는 또 다른 접근법이다. 빠르고 최적화되게 실행할 수 있자만, 기본 목표는 모듈화된 코드를 만드는 것이다. 그 일환으로, script 태그를 위해 URL 대신에 모듈 ID를 사용하는 것을 권장한다.

RequireJS loads all code relative to a baseUrl. The baseUrl is normally set to the same directory as the script used in a data-main attribute for the top level script to load for a page. The data-main attribute is a special attribute that require.js will check to start script loading. This example will end up with a baseUrl of scripts:

```html
<!--This sets the baseUrl to the "scripts" directory, and
    loads a script that will have a module ID of 'main'-->
<script data-main="scripts/main.js" src="scripts/require.js"></script>
```

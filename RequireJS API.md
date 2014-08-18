REQUIREJS API
=====================

[http://pebf.tistory.com/77](http://pebf.tistory.com/77) 번역을 사용했습니다.

이해를 돕기위해 약간의 의역이 존재합니다. 위 주소에서 가져온 번역을 제외하고는 발번역입니다. 참고하세요.

##### Table of Contents

1. [사용법](#Usage)  
  1. [자바스크립트 파일 로딩](#Load_JavaScript_Files)  
  1. [data-main 진입점](#data-main_Entry_Point)  
  1. [모듈 정의하기](#Define_a_Module)  
    1. [단순한 Name/Value 쌍](#Simple_Name_Value_Pairs)  
    1. [함수 정의](#Definition_Functions)  
    1. [디펜던시가 있는 함수 정의](#Definition_Functions_with_Dependencies)  
    1. [모듈을 함수로 정의하기](#Define_a_Module_as_a_Function)  
    1. [모듈을 단순화된 CommonJS 래퍼로 정의하기](#Define_a_Module_with_Simplified_CommonJS_Wrapper)  
    1. [모듈을 이름으로 정의하기](#Define_a_Module_with_a_name)  
    1. [모듈 관련 기타사항](#Other_Module_Notes)  
    1. [순환 디펜던시](#Circular_Dependencies)  
    1. [JSONP 서비스 디펜던시 작성하기](#Specify_a_JSONP_Service_Dependency)  
    1. [모듈 정의하지 않기](#Undefining_a_Module)  
1. [Mechanics](#Mechanics)  
1. [설정 옵션](#Configuration_Options)  
1. [Advanced Usage](#Advanced_Usage)  
  1. [Loading Modules from Packages](#Loading_Modules_from_Packages)  
  1. [Multiversion Support](#Multiversion_Support)  
  1. [Loading Code After Page Load](#Loading_Code_After_Page_Load)  
  1. [Web Worker Support](#Web_Worker_Support)  
  1. [Rhino Support](#Rhino_Support)  
  1. [Handling Errors](#Handling_Errors)  
1. [Loader Plugins](#Loader_Plugins)  
  1. [Specify a Text File Dependency](#Specify_a_Text_File_Dependency)  
  1. [Page Load Event Support/DOM Ready](#Page_Load_Event_Support_DOM_Ready)  
  1. [국제화 번들 정의하기](#Define_an_I18N_Bundle)  

<a name="Usage">
## 사용법

<a name="Load_JavaScript_Files">
### 자바스크립트 파일 로딩

RequireJS takes a different approach to script loading than traditional &lt;script&gt; tags. While it can also run fast and optimize well, the primary goal is to encourage modular code. As part of that, it encourages using **module IDs** instead of URLs for script tags.


RequireJS는 전통적인 &lt;script&gt; 태그를 사용한 스크립트 로딩 방법과 다른 접근 방법을 취하고 있습니다. 이 방법은 빠르게 동작하고, 잘 최적화될 수 있는 동시에, 코드의 모듈화를 증진시키는 것을 목표로 합니다. 그 일부를 보여주는 것이 스크립트 태그에 대해 URL을 사용하는 대신에 **모듈ID**를 사용하는 것을 권장한다.

RequireJS loads all code relative to a baseUrl. The baseUrl is normally set to the same directory as the script used in a data-main attribute for the top level script to load for a page. The data-main attribute is a special attribute that require.js will check to start script loading. This example will end up with a baseUrl of **scripts**:

RequireJS는 baseUrl에 포함된 모든 코드를 불러옵니다. baseUrl은 일반적으로 한 페이지에 로딩되는 최상위 스크립트의 data-main 어트리뷰트에서 쓰이는 스크립트와 같은 디렉토리로 설정됩니다. data-main 어트리뷰트은 require.js가 스크립트 로딩을 시작할 지 확인하는 특별한 어트리뷰트입니다. 다음 예시는 스크립트의 baseUrl을 가지고 있습니다.

```html
<!--This sets the baseUrl to the "scripts" directory, and
    loads a script that will have a module ID of 'main'-->
<script data-main="scripts/main.js" src="scripts/require.js"></script>
```

Or, baseUrl can be set manually via the RequireJS config. If there is no explicit config and data-main is not used, then the default baseUrl is the directory that contains the HTML page running RequireJS.

baseUrl은 RequireJS config를 통해 직접 설정할 수도 있습니다. 설정이 충돌되는 것이 없고 data-main이 사용되지 않았다면, 디폴트 baseUrl은 RequireJS가 실행되는 HTML 페이지를 포함하는 디렉토리입니다. 

RequireJS also assumes by default that all dependencies are scripts, so it does not expect to see a trailing ".js" suffix on module IDs. RequireJS will automatically add it when translating the module ID to a path. With the paths config, you can set up locations of a group of scripts. All of these capabilities allow you to use smaller strings for scripts as compared to traditional &lt;script&gt; tags.

RequireJS는 디폴트로 모든 디펜던시가 스크립트라고 가정하고 있습니다. 그렇기 때문에 ".js" 를 모듈ID에 붙여줄 필요는 없습니다. RequireJS는 모듈ID를 해석할 때 자동적으로 패스에 ".js"를 붙여줍니다. paths 설정을 통해서, 스크립트 모음의 위치를 설정해줄 수도 있습니다. 이러한 특징들은 전통적인 &lt;script&gt; 태그와 비교할 때 스크립트 사용 시 문자열을 더 적게 사용하게 합니다.

There may be times when you do want to reference a script directly and not conform to the "baseUrl + paths" rules for finding it. If a module ID has one of the following characteristics, the ID will not be passed through the "baseUrl + paths" configuration, and just be treated like a regular URL that is relative to the document:

RequireJS를 사용하면서 스크립트를 참조할 때 파일을 찾기 위해 "baseUrl + paths" 방식을 사용하지 않고 직접 참조하고 싶을 때가 있을 것입니다. 만약 모듈ID가 다음의 특정 문자 중 하나를 가지고 있다면, ID는 "baseUrl + paths" 설정을 무시하고, 문서에 관한 일반적인 URL로 취급할 것입니다:

* Ends in ".js".
* Starts with a "/".
* Contains an URL protocol, like "http:" or "https:".

- ".js"로 끝나는 경우 
- "/"로 시작하는 경우  
- "http:" 나 "https:"와 같은 URL 프로토콜을 포함하고 있는 경우

In general though, it is best to use the baseUrl and "paths" config to set paths for module IDs. By doing so, it gives you more flexibility in renaming and configuring the paths to different locations for optimization builds.

일반적인 관점에서, 모듈ID의 paths를 설정하기 위해 baseUrl과 "paths" 설정을 사용하는 것이 최선입니다. 이렇게 함으로써 빌드를 최적화하기 위해 다른 위치로 패스를 설정하는 것과 이름을 바꾸는 것에 대해서 더 많은 유연성을 가지게 됩니다.

Similarly, to avoid a bunch of configuration, it is best to avoid deep folder hierarchies for scripts, and instead either keep all the scripts in baseUrl, or if you want to separate your library/vendor-supplied code from your app code, use a directory layout like this:

유사하게, 설정 폭탄을 피하기 위한 최선은, 스크립트에 대해 깊은 폴더 구조를 피하고, baseUrl안에 모든 스크립트를 넣어두는 것입니다. 만약 라이브러리나 밴더가 공급한 코드를 당신의 애플리케이션의 코드와 분리하고 싶다면, 디렉토리 레이아웃을 다음과 같이 잡을 수 있습니다:
```
www/
	index.html
	js/
		app/
			sub.js
		lib/
			jquery.js
			canvas.js
		app.js
```

in index.html:

index.html 내부
```html
<script data-main="js/app.js" src="js/require.js"></script>
```
and in app.js:

app.js 내부
```javascript
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
```
Notice as part of that example, vendor libraries like jQuery did not have their version numbers in their file names. It is recommended to store that version info in a separate text file if you want to track it, or if you use a tool like volo, it will stamp the package.json with the version information but keep the file on disk as "jquery.js". This allows you to have the very minimal configuration instead of having to put an entry in the "paths" config for each library. For instance, configure "jquery" to be "jquery-1.7.2".

예제를 보면 jQuery와 같은 밴더 라이브러리는 파일명에 버전 넘버를 가지고 있지 않은 것을 알 수 있습니다. 만약 버전 넘버를 따로 트래킹하고 싶다면 별도의 텍스트파일안에 저장하거나, volo 같은 툴을 사용하고 있다면, volo가 버전 정보를 package.json기록할 것입니다. 파일은 "jquery.js"로 유지하는 것이 좋습니다. 이것을 통해 각 라이브러리를 위해 "paths" 설정을 넣어주는 대신에 최소한의 설정을 유지할 수 있습니다. 예를 들어 "jquery-1.7.2"를 사용하기 위해 "jquery"라고 추가로 설정을 하지 않을 수 있도록 말입니다. 

Ideally the scripts you load will be modules that are defined by calling define(). However, you may need to use some traditional/legacy "browser globals" scripts that do not express their dependencies via define(). For those, you can use the shim config. To properly express their dependencies.

이상적으로는 로딩되는 스크립트는 define() 선언에 의해 정의되어야합니다. 그렇지만 의존성을 deinfe()을 통해 표현하지 않는 전통적/레가시적인 "브라우저 전역" 스크립트를 사용할 필요가 있을 수 있습니다. 이러한 경우에 대해서 "shim" 설정을 사용할 수 있습니다. 이를 통해 적절하게 그들의 디펜던시들을 표현할 수 있습니다.

If you do not express the dependencies, you will likely get loading errors since RequireJS loads scripts asynchronously and out of order for speed.

만약 디펜던시를 적어주지 않는다면, RequireJS가 스크립트를 비동기적으로 불러올 때 에러가 발생되고 엉망이 될 것 입니다.

<a name="data-main_Entry_Point">
### data-main Entry Point(data-main 진입점)
The data-main attribute is a special attribute that require.js will check to start script loading:

data-main 어트리뷰트는 RequireJS가 스크립트 로딩을 시작할 때 체크하는 특별한 어트리뷰트입니다:
```html
<!--when require.js loads it will inject another script tag
    (with async attribute) for scripts/main.js-->
<script data-main="scripts/main" src="scripts/require.js"></script>
```
You will typically use a data-main script to set configuration options and then load the first application module. Note: the script tag require.js generates for your data-main module includes the async attribute. This means that **you cannot assume that the load and execution of your data-main script will finish prior to other scripts referenced later in the same page.**

일반적으로 data-main 스크립트를 설정 옵션을 세팅하기 위해 사용한 후에 어플리케이션의 첫번째 모듈을 불러올 것 입니다. Note : require.js가 생성하는 data-main 모듈에 대한 스크립트 태그는 비동기 어트리뷰트를 포함하고 있습니다. **이것은 data-main 스크립트의 로딩과 실행이 같은 페이지 안의 참조되는 다른 스크립트보다 일찍 수행될 것이라고 가정할 수 없다는 것을 의미합니다.**

For example, this arrangement will fail randomly when the require.config path for the 'foo' module has not been set prior to it being require()'d later:

예를 들어, 'foo' 모듈에 대한 'foo'모듈이 require()되기 전에 require.config path가 위치하지 않았을 때 다음과 같이 나열한다면 랜덤하게 실패하는 경우가 생길 것입니다:
```html
<script data-main="scripts/main" src="scripts/require.js"></script>
<script src="scripts/other.js"></script>
```
```javascript
// contents of main.js:
require.config({
    paths: {
        foo: 'libs/foo-1.1.3'
    }
});
```
```javascript
// contents of other.js:

// This code might be called before the require.config() in main.js
// has executed. When that happens, require.js will attempt to
// load 'scripts/foo.js' instead of 'scripts/libs/foo-1.1.3.js'
require( ['foo'], function( foo ) {

});
```

<a name="Define_a_Module">
### Define a Module(모듈 정의하기)

A module is different from a traditional script file in that it defines a well-scoped object that avoids polluting the global namespace. It can explicitly list its dependencies and get a handle on those dependencies without needing to refer to global objects, but instead receive the dependencies as arguments to the function that defines the module. Modules in RequireJS are an extension of the Module Pattern, with the benefit of not needing globals to refer to other modules.

모듈은 전통적인 스크립트 파일과는 다릅니다. 모듈 전역 네임스페이스를 오염시키지 않으며 스코프가 형성되어 있는 객체입니다. 이것은 디펜던시들의 목록을 명쾌하게 보여줄 수 있으며 전역 객체를 참조할 필요없이 디펜던시를 조작할 수 있도록 해주며. 모듈을 정의하는 함수의 인자로 디펜던시를 넘겨줍니다. RequireJS에서의 모듈은 모듈패턴의 확장입니다. 다른 모듈을 참조하기 위해 전역객체를 필요로 하지 않는 이점을 더했습니다.

The RequireJS syntax for modules allows them to be loaded as fast as possible, even out of order, but evaluated in the correct dependency order, and since global variables are not created, it makes it possible to load multiple versions of a module in a page.

모듈에 대한 RequireJS 문법을 통해 가능한한 빠르게 모듈을 로딩할 수 있으며, 순서가 정해져있지 않더라도 이를 정확한 디펜던시 순서로 정렬하며, 전역 변수들이 생성되지 않기 때문에 다양한 버전의 모듈을 한 페이지에 불러오는 것을 가능하게 합니다.

(If you are familiar with or are using CommonJS modules, then please also see CommonJS Notes for information on how the RequireJS module format maps to CommonJS modules).

(만약 CommonJS 모듈을 사용하는 것에 익숙하다면, RequireJS 모듈이 CommonJS 모듈 형식에 어떻게 매칭되는지에 관한 CommonJS Notes도 함께 보시기 바랍니다.)

There should only be **one** module definition per file on disk. The modules can be grouped into optimized bundles by the optimization tool.

디스크 상의 파일 하나 당 오직 **하나의 모듈만 정의**되어야 합니다. 모듈은 최적화 툴에 의해 최적화된 묶음으로 그루핑될 수 있습니다. 

<a name="Simple_Name_Value_Pairs">
#### Simple Name/Value Pairs(단순한 Name/Value 쌍)

If the module does not have any dependencies, and it is just a collection of name/value pairs, then just pass an object literal to define():

모듈이 어떤 디펜던시도 가지고 있지 않고, name/value 쌍으로 이루어져 있다면 객체 리터럴을 define에 넘겨줄 수 있습니다:
```javascript
//Inside file my/shirt.js:
define({
    color: "black",
    size: "unisize"
});
```

<a name="Definition_Functions">
#### Definition Functions(함수 정의)

If the module does not have dependencies, but needs to use a function to do some setup work, then define itself, pass a function to define():

만약 모듈이 디펜던시를 가지고 있지 않지만, 작업을 위해 함수를 사용할 필요가 있다면 define한 후에 define() 내에 함수를 추가합니다:
```javascript
//my/shirt.js now does setup work
//before returning its module definition.
define(function () {
    //Do setup work here

    return {
        color: "black",
        size: "unisize"
    }
});
```
<a name="Definition_Functions_with_Dependencies">
#### Definition Functions with Dependencies(디펜던시가 있는 함수 정의)
If the module has dependencies, the first argument should be an array of dependency names, and the second argument should be a definition function. The function will be called to define the module once all dependencies have loaded. The function should return an object that defines the module. The dependencies will be passed to the definition function as function arguments, listed in the same order as the order in the dependency array:

만약 모듈이 디펜던시가 있다면 첫번째 인자는 디펜던시들의 이름의 배열이어야하고, 두번째 인자는 함수 정의여야 합니다. 함수는 모든 디펜던시가 로딩되었을 때 모듈에 대한 define을 호출할 것입니다. 함수는 모듈을 정의한 객체를 반환해야합니다. 디펜던시는 정의된 함수에 함수 인자의 형태로 넘어갈 것이며, 디펜던시 배열의 순서와 같은 순서로 들어갈 것입니다:
```javascript
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
```
In this example, a my/shirt module is created. It depends on my/cart and my/inventory. On disk, the files are structured like this:

이 예제에서 my/shirt 모듈이 생성되었습니다. 이 모듈은 my/cart와 my/inventory에 의존하고 있습니다. 디스크에서 파일구조는 다음과 같이 이루어져 있습니다:
* my/cart.js
* my/inventory.js
* my/shirt.js

The function call above specifies two arguments, "cart" and "inventory". These are the modules represented by the "./cart" and "./inventory" module names.

위의 함수 호출은 두개의 인자를 기술하고 있습니다. "cart"와 "inventory". 이 모듈들은 "./cart", "./inventory" 모듈명으로 표시되고 있습니다.

The function is not called until the my/cart and my/inventory modules have been loaded, and the function receives the modules as the "cart" and "inventory" arguments.

함수는 my/cart와 my/inventory 두개의 모듈이 로딩될 때까지 호출되지 않습니다. 그리고 함수는 모듈들을 "cart"와 "inventory" 인자를 받습니다.

Modules that define globals are explicitly discouraged, so that multiple versions of a module can exist in a page at a time (see **Advanced Usage**). Also, the order of the function arguments should match the order of the dependencies.

모듈은 전역 변수를 정의하는 것을 분명하게 권장하지 않습니다. 모듈의 복수 버전은 한 페이지 내에 동시에 존재할 수 있습니다.(**응용편을 참조바랍니다.**) 또한 함수 인자의 순서는 디펜던시들의 순서와 매칭되어야 합니다.

The return object from the function call defines the "my/shirt" module. By defining modules in this way, "my/shirt" does not exist as a global object.

함수 호출을 통해 반환되는 객체는 "my/shirt" 모듈에서 정의합니다. 이런 방법으로 모듈을 정의하게되면, "my/shirt"는 전역 객체 내에 존재하지 않게됩니다.

<a name="Define_a_Module_as_a_Function">
#### Define a Module as a Function(모듈을 함수로 정의하기)
Modules do not have to return objects. Any valid return value from a function is allowed. Here is a module that returns a function as its module definition:

모듈은 따로 반환 객체를 가지고 있지 않습니다. 함수의 어떤 유효한 반환값도 허용됩니다. 모듈 정의로써 함수를 반환하는 모듈의 예가 다음에 나와있습니다:
```javascript
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
```
<a name="Define_a_Module_with_Simplified_CommonJS_Wrapper">
#### Define a Module with Simplified CommonJS Wrapper(모듈을 단순화된 CommonJS 래퍼로 정의하기)
If you wish to reuse some code that was written in the traditional CommonJS module format it may be difficult to re-work to the array of dependencies used above, and you may prefer to have direct alignment of dependency name to the local variable used for that dependency. You can use the simplified CommonJS wrapper for those cases:

만약 예전의 CommonJS 모듈 형식으로 작성된 코드를 재사용하고 싶을 때, 위에 사용된 디편던시들의 배열의 형태로 재작업하는 것은 어려울 것이며, 아마도 의존성을 위해 디펜던시들의 이름을 지역변수의 형태로 나열하는 것을 선호할 것 입니다. 이러한 경우에 simplified CommonJS wrapper를 사용할 수가 있습니다:
```javascript
define(function(require, exports, module) {
    var a = require('a'),
        b = require('b');

   //Return the module value
   return function () {};
});
```
This wrapper relies on Function.prototype.toString() to give a useful string value of the function contents. This does not work on some devices like the PS3 and some older Opera mobile browsers. Use the optimizer to pull out the dependencies in the array format for use on those devices.

이 wrapper는 함수의 내용에 관한 유용한 문자열 값을 주기 위해 Functon.prototype.toString()에 의존하고 있습니다. 이것은 PS3와 같은 몇몇 디바이스와 오래된 Opera 모바일 브라우저에서 동작을 하지 않습니다. 이런 기기들에서 사용하기 위해 optimizer 사용할 수 있습니다. 

More information is available on the CommonJS page, and in the in the "Sugar" section in the Why AMD page.

CommonJS 페이지의 Why AMD 페이지의  "Sugar" 섹션에서 더 많은 정보를 찾아볼 수 있습니다.

<a name="Define_a_Module_with_a_Name">
#### Define a Module with a Name(모듈을 이름으로 정의하기)
You may encounter some define() calls that include a name for the module as the first argument to define():

당신은 define() 호출 시에 define()의 첫번째 인자로 모듈의 이름을 포함되어 있는 것을 보게 될 수도 있습니다:
```javascript
//Explicitly defines the "foo/title" module:
define("foo/title",
	["my/cart", "my/inventory"],
	function(cart, inventory) {
		//Define foo/title object in here.
	}
);
```
These are normally generated by the optimization tool. You can explicitly name modules yourself, but it makes the modules less portable -- if you move the file to another directory you will need to change the name. It is normally best to avoid coding in a name for the module and just let the optimization tool burn in the module names. The optimization tool needs to add the names so that more than one module can be bundled in a file, to allow for faster loading in the browser.

이것은 일반적으로 optimizer tool에서 생성되는 형태입니다. 당신은 모듈의 이름을 명확하게 지을 수 있지만, 이것은 모듈의 이동성을 떨어트립니다. -- 만약 당신이 파일을 다른 디렉토리로 옮기게 되면 당신은 이름을 변경해야 합니다. 일반적인 경우에 모듈의 이름을 코딩하는 것은 피하고, 최적화툴로 모듈의 이름을 작성하도록 하는 것이 최선입니다. optimizer tool에서는 브라우저의 로딩 속도를 높이기 위해, 하나 이상의 모듈이 하나의 파일로 합쳐질 수 있도록 이름을 더해주는 것을 필요로합니다. 

<a name="Other_Module_Notes">
#### Other Module Notes(모듈 관련 기타사항)
**One module per file.**: Only one module should be defined per JavaScript file, given the nature of the module name-to-file-path lookup algorithm. Multiple modules will be grouped into optimized files by the optimization tool, but you should only use the optimization tool to place more than one module in a file.

**파일 하나 당 하나의 모듈** : 자바스크립트 파일 하나당 오직 하나의 모듈을 써야하며, name-to-file-path lookup 알고리즘 유형을 따라야합니다. 복수의 모듈은 최적화 도구에 의해 최적화된 파일로 그루핑될 것이다. 그렇지만 하나의 이상의 모듈을 하나의 파일 안에 넣어야 할 때만 최적화 도구를 사용해야 합니다.

**Relative module names inside define()**: For require("./relative/name") calls that can happen inside a define() function call, be sure to ask for "require" as a dependency, so that the relative name is resolved correctly:

**define() 내부에서의 상대적인 모듈명 사용** : require("./relative/name") 호출은 define() 함수 선언 내부에서 일어날 수 있으며, 반드시 "require"를 디펜던시로서 찾고, 상대적인 이름을 통해서 이를 불러올 수 있습니다:
```javascript
define(["require", "./relative/name"], function(require) {
	var mod = require("./relative/name");
});
```
Or better yet, use the shortened syntax that is available for use with translating CommonJS modules:

CommonJS 모듈을 해석을 통해 단축된 문법을 사용할 수 있다면 더욱 좋습니다:
```javascript
define(function(require) {
    var mod = require("./relative/name");
});
```
This form will use Function.prototype.toString() to find the require() calls, and add them to the dependency array, along with "require", so the code will work correctly with relative paths.

이 형태는 require() 호출을 찾기 위해 Function.prototype.toString()을 사용합니다. 그리고 그것들을 "require"를 따라 디펜던시 배열에 추가합니다. 코드는 상대경로에 의해 정확하게 동작할 것 입니다. 

Relative paths are really useful if you are creating a few modules inside a directory, so that you can share the directory with other people or other projects, and you want to be able to get a handle on the sibling modules in that directory without having to know the directory's name.

상대경로는 디렉토리 안에 적은 모듈을 생성할 때 매우 유용합니다. 이를 통해 당신은 디렉토리를 다른 사람이나 다른 프로젝트와 공유할 수 있습니다. 그리고 디렉토리 내에 있는 다른 모듈을 디렉토리의 이름을 모르는 상태에서도 관리할 수 있을 것 입니다.

**Generate URLs relative to module**: You may need to generate an URL that is relative to a module. To do so, ask for "require" as a dependency and then use require.toUrl() to generate the URL:

**모듈에 대한 URL 생성** : 당신은 모듈에 관한 URL을 생성할 필요가 있을 것 입니다. 이를 위해 require를 디펜던시로 찾은 후에 require.toUrl() 사용하여 URL을 생성할 수 있습니다. 
```javascript
define(["require"], function(require) {
    var cssUrl = require.toUrl("./style.css");
});
```
**Console debugging**: If you need to work with a module you already loaded via a require(["module/name"], function(){}) call in the JavaScript console, then you can use the require() form that just uses the string name of the module to fetch it:

**콘솔 디버깅** : 자바스크립트 콘솔에서 require(["module/name"], function(){}) 호출을 통해 이미 로딩된 모듈로 작업을 할 필요가 있다면, 모듈을 불러오기 위해 모듈의 문자열 이름만을 사용하는 require() 형식을 사용할 수 있습니다:
```javascript
require("module/name").callSomeFunction()
```
Note this only works if "module/name" was previously loaded via the async version of require: require(["module/name"]). If using a relative path, like './module/name', those only work inside define.

이 형식은 "module/name"을 require의 비동기 버전을 통해 이미 로드되어있는 경우에만 동작한다는 것을 기억하세요. './module/name'과 같이 상대경로를 사용하는 방식은 define 내부에서만 동작합니다.

<a name="Circular_Dependencies">
#### Circular Dependencies(순환 디펜던시)
If you define a circular dependency ("a" needs "b" and "b" needs "a"), then in this case when "b"'s module function is called, it will get an undefined value for "a". "b" can fetch "a" later after modules have been defined by using the require() method (be sure to specify require as a dependency so the right context is used to look up "a"):

만약 당신이 순환 디펜던시를 정의했다면 (a는 b가 필요하고, b는 a가 필요한) 이 경우에 "b" 모듈 함수가 호출된다면, 이것은 "a"에 대해서 undefined 값을 가지게 될 것 입니다. "b"는 "a"를 모듈이 require() 메소드에 의해 정의된 이후에 가져올 수 있습니다. (반드시 require를 디펜던시로 명시해서 올바른 context 내에서 "a"를 찾아야 합니다.):
```javascript
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
```
Normally you should not need to use require() to fetch a module, but instead rely on the module being passed in to the function as an argument. Circular dependencies are rare, and usually a sign that you might want to rethink the design. However, sometimes they are needed, and in that case, use require() as specified above.

일반적인 경우에는 모듈을 불러오기 위해서 require()를 사용할 필요는 없으며, 함수에 인자로 모듈을 넘겨주면 됩니다. 순환 디펜던시는 드문 경우이며, 디자인에 대해서 다시 생각해봐야 한다는 신호이기도 합니다. 그러나 때때로 필요할 경우가 있으며, 그런 경우에는 require()를 위에 기술해 놓은 것 처럼 사용해야합니다. 

If you are familiar with CommonJS modules, you could instead use exports to create an empty object for the module that is available immediately for reference by other modules. By doing this on both sides of a circular dependency, you can then safely hold on to the the other module. This only works if each module is exporting an object for the module value, not a function:

CommonJS 모듈이 친숙하다면, exports를 사용할 수도 있습니다. 이를 다른 모듈들에 의한 참조에 바로 사용될 수 있는 현재 모듈을 가리키는 빈 객체로 사용이 가능합니다. 이것을 양쪽 모듈에 적용하면, 다른 모듈을 안전하게 기다리게 할 수 잇습니다. 이것은 모듈값으로 함수가 아닌 객체를 exporting할 때에만 동작합니다:
```javascript
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
```
Or, if you are using the dependency array approach, ask for the special 'exports' dependency:

또는 디펜던시 배열 접근 방법을 사용하고 있다면 특별한 'exports' 디펜던시를 찾을 수도 있습니다:
```javascript
//Inside b.js:
define(['a', 'exports'], function(a, exports) {
    //If "a" has used exports, then we have a real
    //object reference here. However, we cannot use
    //any of "a"'s properties until after "b" returns a value.

    exports.foo = function () {
        return a.bar();
    };
});
```

<a name="Specify_a_JSONP_Service_Dependency">
#### Specify a JSONP Service Dependency(JSONP 서비스 디펜던시 작성하기)
JSONP is a way of calling some services in JavaScript. It works across domains and it is an established approach to calling services that just require an HTTP GET via a script tag.

JSONP는 자바스크립트에서 서비스를 호출하는 방법중 하나입니다. 이것은 도메인을 통해서 작동하고, script 태그를 통해서 http get을 요청하는 서비스를 연결하는 방법입니다.

To use a JSONP service in RequireJS, specify "define" as the callback parameter's value. This means you can get the value of a JSONP URL as if it was a module definition.

RequireJS에서 JSONP 서비스를 사용하기 위해, 파라미터의 값으로써 "define"에서 작성합니다.

Here is an example that calls a JSONP API endpoint. In this example, the JSONP callback parameter is called "callback", so "callback=define" tells the API to wrap the JSON response in a "define()" wrapper:

아래는 JSONP API를 호출하는 예제입니다. 이 예제에서, JSONP callback 파라미터는 "callback"이라고 불려지고, "callback=define"은 "define()" 랩퍼에서 JSON 응답을 감싸는 API를 말합니다.
```javascript
require(["http://example.com/api/data.json?callback=define"],
    function (data) {
        //The data object will be the API response for the
        //JSONP data call.
        console.log(data);
    }
);
```
This use of JSONP should be limited to JSONP services for initial application setup. If the JSONP service times out, it means other modules you define via define() may not get executed, so the error handling is not robust.

어플리케이션 초기 설정에서 JSONP 서비스를 제한합니다. 만약 JSONP 서비스가 타임 아웃이 되면, define()을 통해서 정의한 다른 모듈들이 실행할 수 없습니다. 그래서 오류 처리를 할 수 없습니다.

**Only JSONP return values that are JSON objects are supported.** A JSONP response that is an array, a string or a number will not work.

**JSONP로 반환되는 값은 오직 JSON 객체만 지원합니다.** 배열이나 문자열이나 숫자인 JSONP 응답은 작동하지 않습니다.

This functionality should not be used for long-polling JSONP connections -- APIs that deal with real time streaming. Those kinds of APIs should do more script cleanup after receiving each response, and RequireJS will only fetch a JSONP URL once -- subsequent uses of the same URL as a dependency in a require() or define() call will get a cached value.

이 기능은 long-polling JSONP 연결에서는 사용하지 않습니다. long-polling JSONP는 실시간 스트리밍을 다루는 API입니다. 이런 종류의 API는 각각의 응답을 받은 후 많은 script를 정리해야하고, RequireJS는 한번만 JSONP URL을 가져옵니다. 이후에 require()나 define() 호출에서 디펜던시로써 같은 URL을 사용하면 캐시 된 값을 얻을 것입니다.

Errors in loading a JSONP service are normally surfaced via timeouts for the service, since script tag loading does not give much detail into network problems. To detect errors, you can override requirejs.onError() to get errors. There is more information in the Handling Errors section.

JSONP 서비스를 로드할 때 생긴 에러는 보통 서비스에 대한 타임 아웃을 통해서 나타납니다. 로딩중인 script 태그는 네트워크 문제에 대한 세부사항을 주지 않습니다. 에러를 찾기 위해서, requirejs.onError()를 override할 수 있습니다. 에러 핸들링 섹션에서 더 많은 정보가 있습니다.

<a name="Undefining_a_Module">
#### Undefining a Module(모듈 정의하지 않기)
There is a global function, **requirejs.undef()**, that allows undefining a module. It will reset the loader's internal state to forget about the previous definition of the module.

전역함수인 requirejs.undef()는 정의되지 않는 모듈을 허용하게 한다. 이 함수는 모듈의 이전의 정의를 잊도록 로더의 내부 상태를 리셋합니다.

**However**, it will not remove the module from other modules that are already defined and got a handle on that module as a dependency when they executed. So it is really only useful to use in error situations when no other modules have gotten a handle on a module value, or as part of any future module loading that may use that module. See the errback section for an example.

그러나 이것은 다른 모듈에서 이미 정의되어 그들이 실행될 때 관리하게된 모듈을 제거하지는 못합니다. 이것은 사용될 미래의 모듈 로딩 또는 모듈값을 관리하는 다른 모듈이 없는 경우에 에러가 발생한 경우에만 유용합니다. errback 섹션에 있는 예제를 참고하기 바랍니다.

If you want to do more sophisticated dependency graph analysis for undefining work, the semi-private onResourceLoad API may be helpful.

만약 당신이 정의되지 않은 작업에 대해 정교한 디펜던시 그래프 분석을 하길 원한다면 semi-private onResourceLoadAPI가 유용할 것 입니다.

<a name="Mechanics">
## MECHANICS
RequireJS loads each dependency as a script tag, using head.appendChild().

RequireJS는 디펜던시를 head.appendChild()를 통해 스크립트 태그 형태로 불러들입니다.

RequireJS waits for all dependencies to load, figures out the right order in which to call the functions that define the modules, then calls the module definition functions in the right order.

RequireJS 모든 디펜던시가 로딩되기를 기다린 후에, 모듈을 정의한 함수를 호출하기 위한 올바른 순서를 계산합니다. 그런 후에 모듈을 정의한 함수를 올바른 순서로 호출합니다.

Using RequireJS in a server-side JavaScript environment that has synchronous loading should be as easy as redefining require.load(). The build system does this, the require.load method for that environment can be found in build/jslib/requirePatch.js.

동기적 로딩을 하는 서버-사이드 환경에서 RequireJS를 사용하는 것은 require.load()를 재정의하면 매우 쉬워집니다. 빌드 시스템이 이를 수행하면, 서버사이드 환경을 위한 require.load 메소드는 build/jslib/requirePatch.js를 찾을 수 있을 것입니다. 

In the future, this code may be pulled into the require/ directory as an optional module that you can load in your env to get the right load behavior based on the host environment.

향후에, 이 코드는 부가적인 모듈로서 require/ 디렉토리로 가져올 것입니다. 이를 통해 당신의 환경에 이것을 불러와 호스트 환경에 기초한 올바른 로딩 방법을 사용할 수 있게될 것 입니다.

<a name="Configuration_Options">
## CONFIGURATION OPTIONS(설정 옵션)
When using require() in the top-level HTML page (or top-level script file that does not define a module), a configuration object can be passed as the first option:

탑-레벨 HTML 페이지(또는 모듈을 정의하지 않는 탑-레벨스크립트 파일)에서 require()를 사용할 때, 설정 객체는 첫번째 옵션으로 넘겨지게 됩니다. 
```html
<script src="scripts/require.js"></script>
<script>
  require.config({
    baseUrl: "/another/path",
    paths: {
        "some": "some/v1.0"
    },
    waitSeconds: 15
  });
  require( ["some/module", "my/module", "a.js", "b.js"],
    function(someModule,    myModule) {
        //This function will be called when all the dependencies
        //listed above are loaded. Note that this function could
        //be called before the page is loaded.
        //This callback is optional.
    }
  );
</script>
```
You may also call require.config from your data-main Entry Point, but be aware that the data-main script is loaded asynchronously. Avoid other entry point scripts which wrongly assume that data-main and its require.config will always execute prior to their script loading.

또한 data-main 진입점에서 require.config를 실행할 수도 있습니다. 그렇지만 data-main 스크립트는 비동기적으로 불러온다는 것을 알고 있어야 합니다. data-main과 require.config가 항상 스크립트 로딩 전에 로딩된다고 잘못된 가정을 하는 다른 진입점의 스크립트는 피하는 것이 좋습니다. 

Also, you can define the config object as the global variable require before require.js is loaded, and have the values applied automatically. This example specifies some dependencies to load as soon as require.js defines require():

또한 당신은 설정 객체를 require.js가 로딩되기 전에 전역 변수인 require로 정의할 수 있습니다. 이렇게하면 값은 자동적으로 적용되게 됩니다. 다음 예제는 require.js가 require()를 정의하자마자 불러올 몇가지 디펜던시를 정의하고 있습니다. 
```html
<script>
    var require = {
        deps: ["some/module1", "my/module2", "a.js", "b.js"],
        callback: function(module1, module2) {
            //This function will be called when all the dependencies
            //listed above in deps are loaded. Note that this
            //function could be called before the page is loaded.
            //This callback is optional.
        }
    };
</script>
<script src="scripts/require.js"></script>
```
**Note**: It is best to use var require = {} and do not use window.require = {}, it will not behave correctly in IE.

**주의사항**: var require = {} 와 같이 사용한는 것이 최선이며, window.require = {}와 같이 사용하지 않는 것이 좋습니다. 뒤의 방법은 IE에서 제대로 동작하지 않을 것 입니다.

There are some patterns for separating the config from main module loading.

설정을 메인 모듈 로딩과 분리하는 몇가지 패턴이 있습니다.

Supported configuration options:

제공되는 설정 옵션:

**baseUrl**: the root path to use for all module lookups. So in the above example, "my/module"'s script tag will have a src="/another/path/my/module.js". baseUrl is **not** used when loading plain .js files (indicated by a dependency string starting with a slash, has a protocol, or ends in .js), those strings are used as-is, so a.js and b.js will be loaded from the same directory as the HTML page that contains the above snippet.

**baseUrl**: 모든 모듈 검색에 사용될 루트 패스입니다. 위의 예제에서 "my/module"의 스크립트 태그는 src="/another/path/my/modules.js"를 가지게 될 것 입니다. baseUrl은 .js파일(슬래시로 시작하거나, 프로토콜은 가지고 있거나, .js로 끝나는 디펜던시 문자열로 표현되는)을 불러올 때는 사용되지 않습니다. 이러한 문자열들은 그대로 사용되며, 그렇기 때문에 a.js와 b.js는 위에 코드를 포함하고 있는 HTML페이지와 같은 디렉토리에서 불러오게 됩니다. 

If no baseUrl is explicitly set in the configuration, the default value will be the location of the HTML page that loads require.js. If a **data-main** attribute is used, that path will become the baseUrl.

설정에 명확한 baseUrl이 보이지 않는다면, 기본값은 require.js를 불러오는 HTML 페이지의 위치가 될 것 입니다. 만약 **data-main** 어트리뷰트가 사용된다면 그 위치가 baseUrl이 될 것 입니다.

The baseUrl can be a URL on a different domain as the page that will load require.js. RequireJS script loading works across domains. The only restriction is on text content loaded by text! plugins: those paths should be on the same domain as the page, at least during development. The optimization tool will inline text! plugin resources so after using the optimization tool, you can use resources that reference text! plugin resources from another domain.

baseUrl은 require.js가 로드될 페이지와 다른 도메인의 URL이 될 수 있습니다. RequireJS 스크립트 로딩은 domain을 통해서 작동한다. 유일한 제한은 text! 플러그인에 의해서 text content를 로딩할 때 입니다. 이 경로는 적어도 개발하는 동안에는 페이지와 같은 동일한 domain에 있어야 한다. 최적화 도구는 text! 플러그인이 내부에 있어서 최적화 도구를 사용 후에는 다른 도메인에서 text! 플러그인 리소스를 참조하는 리소스를 사용할 수 있다.

**paths**: path mappings for module names not found directly under baseUrl. The path settings are assumed to be relative to baseUrl, unless the paths setting starts with a "/" or has a URL protocol in it ("like http:"). Using the above sample config, "some/module"'s script tag will be src="/another/path/some/v1.0/module.js".

**paths** : path는 직접적으로 baseUrl의 아래에서 찾을 수 없는 모듈명을 맵핑합니다. 패스를 설정은 path 설정이 "/"로 시작하거나 URL 프로토콜("http:와 같은")을 가지고 있지 않다면 baseUrl과 관련이 있다고 가정합니다. 위의 예제 코드에서, "some/module"의 스크립트 태그는 src="/another/path/some/v1.0/module.js"가 될 것입니다. 

The path that is used for a module name should not include an extension, since the path mapping could be for a directory. The path mapping code will automatically add the .js extension when mapping the module name to a path. If require.toUrl() is used, it will add the appropriate extension, if it is for something like a text template.

모듈명을 위해 사용되는 path는 확장자를 포함하지 않아야합니다. path 맵핑은 디렉토리일 수 있기 때문입니다. path 매핑 코드는 모듈명을 path와 맵핑할 때 자동으로 .js 확장자를 더해줍니다. 만약 require.toUrl이 사용되었다면, 적절한 확장자를 더해줄 것입니다. 이것은 텍스트 템플릿에 관한 것일 수도 있습니다.

When run in a browser, paths fallbacks can be specified, to allow trying a load from a CDN location, but falling back to a local location if the CDN location fails to load.

브라우저에서 실행될 때, paths fallback을 명시할 수 있습니다. 이를 통해 CDN location에서 로딩을 하는 것이 가능해집니다. CDN location의 로딩을 실패했다면 local location에서 로딩할 수 있도록 할 수 있기 때문입니다. 

bundles: Introduced in RequireJS 2.1.10: allows configuring multiple module IDs to be found in another script. Example:

bundles : RequireJS 2.1.10 부터 사용가능합니다: 다른 스크립트에서 찾을 수 있는 복수의 모듈ID 설정을 하도록 해줍니다. 예를 들어 :
```javascript
requirejs.config({
    bundles: {
        'primary': ['main', 'util', 'text', 'text!template.html'],
        'secondary': ['text!secondary.html']
    }
});

require(['util', 'text'], function(util, text) {
    //The script for module ID 'primary' was loaded,
    //and that script included the define()'d
    //modules for 'util' and 'text'
});
```
That config states: modules 'main', 'util', 'text' and 'text!template.html' will be found by loading module ID 'primary'. Module 'text!secondary.html' can be found by loading module ID 'secondary'.

위의 설정 상태: 모듈 'main', 'util', 'text', 'text!template.html'은 로딩 모듈ID를 'primary'로 사용할 수 있을 것입니다. 모듈 'text!secondary,html'는 모듈ID 'secondary'로 사용할 수 있습니다.

This only sets up where to find a module inside a script that has multiple define()'d modules in it. It does not automatically bind those modules to the bundle's module ID. The bundle's module ID is just used for locating the set of modules.

이것은 하나의 스크립트 내부에서 복수의 define()된 모듈이 있을 때 해당 모듈을 찾아야 할 때 설정이 가능합니다. 이것은 자동적으로 bundle의 모듈ID와 모듈들과 바인딩을 시키지 않습니다. bundle의 모듈id는 단순히 모듈의 묶음을 위치시키는 데에 사용됩니다.

Something similar is possible with paths config, but it is much wordier, and the paths config route does not allow loader plugin resource IDs in its configuration, since the paths config values are path segments, not IDs.

paths 설정과 유사하게 사용할 수 있지만, 이것은 매우 장황하며, paths 설정 루트는 그 설정 내에서 로더 플러그인의 리소스ID를 허용하지 않습니다. 이것은 패스 설정값이 ID들이 아닌 path 부분이기 때문입니다.

bundles config is useful if doing a build and that build target was not an existing module ID, or if you have loader plugin resources in built JS files that should not be loaded by the loader plugin. **Note that the keys and values are module IDs**, not path segments. They are absolute module IDs, not a module ID prefix like paths config or map config. Also, bundle config is different from map config in that map config is a one-to-one module ID relationship, where bundle config is for pointing multiple module IDs to a bundle's module ID.

bundles 설정은 빌드를 실행한 후에 빌드 타켓이 존재하는 모듈ID가 아닐 때나, 빌드된 JS파일 안에 로더 플러그인 리소스를 가지고 있고 그 JS파일은 로더 플러그인에 의해 로드되지 않아야 하는 경우라면 유용합니다. **key/value가 모두 패스가 아니라 모듈ID라는 점에 주목하세요.** 이것들은 절대 모듈ID이여야 하며, paths 설정과 map 설정과 같은 모듈ID 접미사들이 아닙니다. 또한 번들 설정은 map 설정과는 다릅니다. map설정은 일대일 모듈,ID 관계이며 bundle 설정은 복사의 모듈ID들을 bundle의 모듈ID에 설정한다는 점에서 차이가 있습니다. 

**shim**: Configure the dependencies, exports, and custom initialization for older, traditional "browser globals" scripts that do not use define() to declare the dependencies and set a module value.

**shim** : 디펜던시들과 exports, 그리고 디펜던시를 선언을 위해 define()을 사용하지 않는 "브라우저 전역" 스크립트들의 커스텀 초기화들을 설정하고 모듈값을 설정합니다.

Here is an example. It requires RequireJS 2.1.0+, and assumes backbone.js, underscore.js and jquery.js have been installed in the baseUrl directory. If not, then you may need to set a paths config for them:

아래 예제가 있습니다. 이것은 RequireJS 2.1.0+ 이상의 문법이며, backbone.js와 underscore.js 그리고 jquery.js가 baseUrl 디렉토리 안에 있다고 가정합니다. 그렇지 않은 경우라면 이를 위해 paths 설정을 할 필요가 있을 것입니다:
```javascript
requirejs.config({
    //Remember: only use shim config for non-AMD scripts,
    //scripts that do not already call define(). The shim
    //config will not work correctly if used on AMD scripts,
    //in particular, the exports and init config will not
    //be triggered, and the deps config will be confusing
    //for those cases.
    shim: {
        'backbone': {
            //These script dependencies should be loaded before loading backbone.js
            //이 스크립트들의 디펜던시들은 backbone.js가 로딩되기 전에 로딩되어야 합니다.
            deps: ['underscore', 'jquery'],
            //Once loaded, use the global 'Backbone' as the module value.
            ///한 번 로딩이되면 전역 모듈값으로 'Backbone'을 사용합니다.
            exports: 'Backbone'
        },
        'underscore': {
            exports: '_'
        },
        'foo': {
            deps: ['bar'],
            exports: 'Foo',
            init: function (bar) {
                //Using a function allows you to call noConflict for
                //libraries that support it, and do other cleanup.
                //However, plugins for those libraries may still want
                //a global. "this" for the function will be the global
                //object. The dependencies will be passed in as
                //function arguments. If this function returns a value,
                //then that value is used as the module export value
                //instead of the object found via the 'exports' string.
                //Note: jQuery registers as an AMD module via define(),
                //so this will not work for jQuery. See notes section
                //below for an approach for jQuery.
                //noConflict 를 지원하는 라이브러리들에 대해 이를 호출하거나
                //다른 cleanup 작업을 할 수 있는 함수를 사용하는 것이 허용됩니다.
                //그렇지만, 이러한 라이브러리를 위한 플러그인은 여전히 전역이
                //되려고 할 것 입니다. 만약 이 함수가 값을 반환한다면, "exports"
                //문자열을 통해서 발견한 객체 대신에, 이것이 모듈의 exports 값으로
                //사용될 것 입니다.
                //Note : jQurey는 define()을 통해 AMD 모듈로 등록되었다면, 아래의
                //jQuery에 대한 접근 섹션을 참조하세요.        
                return this.Foo.noConflict();
            }
        }
    }
});

//Then, later in a separate file, call it 'MyModel.js', a module is
//defined, specifying 'backbone' as a dependency. RequireJS will use
//the shim config to properly load 'backbone' and give a local
//reference to this module. The global Backbone will still exist on
//the page too.
//그리고 나서 분리된 파일보다 늦게, 'MyModel.js"부르는 모듈이 정의되면
//'backbone'을 디펜던시로 명시합니다.  RequireJS는 shim 설정을
//적절하게 'backbone'을 불러와 모듈 지역 참조에 넘겨주기 위해 사용합니다.
//전역 backbone 역시 여전히 페이지에 남아있습니다.
define(['backbone'], function (Backbone) {
  return Backbone.Model.extend({});
});
```
**기억하세요**
**shim 설정은 non-AMD 스크립트에 대해서만 사용하며 스크립트들은 define() 호출을 미리하지 않았어야 합니다. 만약 AMD 스크립트들에 대해 사용한다면 shim설정은 정상적으로 동작하지 않을 수도 있습니다. 특히 exports와 init 설정이 동작하지 않으며 deps 설정이 꼬이게 될 것 입니다.**

In RequireJS 2.0.*, the "exports" property in the shim config could have been a function instead of a string. In that case, it functioned the same as the "init" property as shown above. The "init" pattern is used in RequireJS 2.1.0+ so a string value for exports can be used for enforceDefine, but then allow functional work once the library is known to have loaded.

RequireJS 2.0.* 버전에서 shim 설정은 문자열 대신에 함수가 될 수도 있습니다. 이런 경우에 위에 보이는 "init" 프로퍼티와 같이 기능합니다. "init" 패턴은 RequireJS 2.1.0+ 버전에서 사용됩니다. 이를 통해 exports에 관한 문자열값은 enforceDefine에 사용될 수 있습니다. 그렇지만 함수적인 작업은 라이브러리가 로딩된 것을 알 수 있을 때 한번만 가능합니다. 

For "modules" that are just jQuery or Backbone plugins that do not need to export any module value, the shim config can just be an array of dependencies:

jQuery와 Backbone 플러그인들처럼 모듈값을 export 할 필요가 없는 모듈들이라면 shim 설정은 디펜던시들의 배열 형태일 수 있습니다:
```javascript
requirejs.config({
    shim: {
        'jquery.colorize': ['jquery'],
        'jquery.scroll': ['jquery'],
        'backbone.layoutmanager': ['backbone']
    }
});
```
Note however if you want to get 404 load detection in IE so that you can use paths fallbacks or errbacks, then a string exports value should be given so the loader can check if the scripts actually loaded (a return from init is **not** used for enforceDefine checking):

만약 IE에서 404 로드를 감지하고 싶다면 paths fallback 또는 errback을 사용하고, 실제로 스크립트가 로딩됐는지 확인할 수 있도록 문자열 exports 값이 loader에 주어져야 합니다(**init 에서 반환되는 값은 enforceDefine 체크에 쓰이지 않습니다.**)
```javascript
requirejs.config({
    shim: {
        'jquery.colorize': {
            deps: ['jquery'],
            exports: 'jQuery.fn.colorize'
        },
        'jquery.scroll': {
            deps: ['jquery'],
            exports: 'jQuery.fn.scroll'
        },
        'backbone.layoutmanager': {
            deps: ['backbone']
            exports: 'Backbone.LayoutManager'
        }
    }
});
```
**Important notes for "shim" config:**

**"shim" 설정에 있어서 주목해야할 부분들:**

* The shim config only sets up code relationships. To load modules that are part of or use shim config, a normal require/define call is needed. Setting shim by itself does not trigger code to load.
* shim 설정은 코드 관계만을 설정해야합니다. shim 설정의 일부인 모듈을 불러오거나 shim 설정을 사용할 때는 일반적인 require/define 호출이 필요합니다. shim 그 자체를 코드를 불러오는 트리거로 사용하지 마십시오. 
* Only use other "shim" modules as dependencies for shimmed scripts, or AMD libraries that have no dependencies and call define() after they also create a global (like jQuery or lodash). Otherwise, if you use an AMD module as a dependency for a shim config module, after a build, that AMD module may not be evaluated until after the shimmed code in the build executes, and an error will occur. The ultimate fix is to upgrade all the shimmed code to have optional AMD define() calls.
* shim된 스크립트에 대한 디펜던시로서만 다른 "shim" 모듈을 사용하십시오. 또는 디펜던시가 없으면서 전역을 생성을 한 후에 define()을 호출하는 AMD 라이브러리(like jQurey 또는 lodash)의 경우에도 사용할 수 있습니다. 반면에 AMD 모듈을 shim 설정 모듈에 대한 디펜던시로 사용한다면 AMD 모듈은 빌드가 실행될 때 shim된 코드가 빌드될 때까지 인식되지 않을 것이며 에러가 발생될 것 입니다. 궁극의 수정은 모든 shim된 코드를 부가적인 AMD define() 호출을 하도록 업그레이드 하는 것입니다.
* If it is not possible to upgrade the shimmed code to use AMD define() calls, as of RequireJS 2.1.11, the optimizer has a wrapShim build option that will try to automatically wrap the shimmed code in a define() for a build. This changes the scope of shimmed dependencies, so it is not guaranteed to always work, but, for example, for shimmed dependencies that depend on an AMD version of Backbone, it can be helpful.
* shim된 코드를 AMD define() 호출을 하도록 업그레이드하는 것이 불가능하다면  RequireJS 2.1.11 버전의 optimizer는 wrapShim build option을 가지고 있습니다. 이것은 빌드 시에 자동적으로 shim된 코드를 define()으로 감싸주는 작업을 합니다. 이것은 shim된 디펜던시의 스코프를 변경시키며, 그로 인해 항상 정상적으로 동작할 것이라는 보장을 할 수 없게 됩니다. 그렇지만 예를 들어 shim된 디펜던시가  Backbone의 AMD 버전에 의존하고 있다면 이것은 도움이 될 수 있습니다. 
* The init function will **not** be called for AMD modules. For example, you cannot use a shim init function to call jQuery's noConflict. See Mapping Modules to use noConflict for an alternate approach to jQuery.
* init 함수는 AMD 모듈에 대해 호출되지 않습니다. 예를 들어 shim init 함수는 jQuery의 noConflict를 호출할 수 없습니다. jQuery에 대한 접근을 위해 noConflict를 사용하기 위해 모듈을 맵핑하기를 살펴보십시오.
* Shim config is not supported when running AMD modules in node via RequireJS (it works for optimizer use though). Depending on the module being shimmed, it may fail in Node because Node does not have the same global environment as browsers. As of RequireJS 2.1.7, it will warn you in the console that shim config is not supported, and it may or may not work. If you wish to suppress that message, you can pass requirejs.config({ suppress: { nodeShim: true }});.
* shim 설정은 node상의 RequireJS에서 AMD 모듈을 실행할 때에는 지원되지 않습니다. (이는 최적화 도구 사용을 위해 동작합니다.) shim된 모듈에 의존성을 가지고 있을 때 Node 안에서는 fail이 발생할 수 있습니다. Node는 브라우저에서와 같은 전역 환경을 제공하지 않기 때문입니다. RequireJS 2.1.7의 경우에는 console에서 shim 설정이 동작할 수도 있고 그렇지 않을 수도 있다는 경고를 줄 것 입니다. 만약 이 메시지를 숨기고 싶다면 requirejs.config({surpress : {nodeShim : true }});

**Important optimizer notes for "shim" config:**

**"sihm" 설정을 위한 최적화도구의 중요한 부분들:**
* You should use the mainConfigFile build option to specify the file where to find the shim config. Otherwise the optimizer will not know of the shim config. The other option is to duplicate the shim config in the build profile.
* 파일을 shim 설정 어디에서 찾을 것인지 명시하기 위해 mainConfigFile 빌드 옵션을 사용해야만 합니다. 그렇지 않으면 optimizer는 shim 설정을 알 수가 없습니다. 다른 옵션은 빌드 프로파일 안의 shim 설정과 중복하게 됩니다.
* Do not mix CDN loading with shim config in a build. Example scenario: you load jQuery from the CDN but use the shim config to load something like the stock version of Backbone that depends on jQuery. When you do the build, be sure to inline jQuery in the built file and do not load it from the CDN. Otherwise, Backbone will be inlined in the built file and it will execute before the CDN-loaded jQuery will load. This is because the shim config just delays loading of the files until dependencies are loaded, but does not do any auto-wrapping of define. After a build, the dependencies are already inlined, the shim config cannot delay execution of the non-define()'d code until later. define()'d modules do work with CDN loaded code after a build because they properly wrap their source in define factory function that will not execute until dependencies are loaded. So the lesson: shim config is a stop-gap measure for non-modular code, legacy code. define()'d modules are better.
* 빌드 시에 CDN과 shim 설정을 섞지 마십시오. 예제 시나리오 : 당신은 jQuery를 CDN으로 부터 로딩합니다. 그렇지만 jQuery에 의존하는 Backbone의 stock 버전과 같은 파일을 shim 설정을 불러오기 위해 shim 설정을 사용할 수 있습니다. 빌드를 할 때 빌드된 파일에서 jQuery를 확인하고 CDN에서 불러오지 않습니다. 그렇지 않으면 Backbone은 빌드된 파일에서 inline하게 될 것이며 이것은 CDN에서 불러온 jQuery가 로딩되기 전에 실행될 것입니다. 이것은 shim 설정이 모든 디펜던시가 로딩될 때까지 파일의 로딩을 딜레이시키기고 다른 define 자동 래핑을 하지 않았기 때문입니다. 빌드 후에 디펜던시는 이미 inline되었고, shim 설정은 define()되지 않은 코드를 이후까지 딜레이 시키지 못합니다. 빌드 후에 define()된 모듈은 CDN으로 로딩된 코드와 작업을 하게 됩니다. 그것들은 적절하게 그들의 소스를 디펜던시가 로딩될 때까지 실행되지 않는 define 팩토리 함수로 래핑해주었기 때문입니다. 그렇다면 여기서 배울 수 있는 점은 무엇일까요 : shim 설정은 모듈화되지않은 코드, 레거시 코드에 대한 stop-gap measure 입니다. define()`된 모듈을 사용하는 것이 더 낫습니다. 
* For local, multi-file builds, the above CDN advice also applies. For any shimmed script, its dependencies **must** be loaded before the shimmed script executes. This means either building its dependencies directly in the buid layer that includes the shimmed script, or loading its dependencies with a require([], function (){}) call, then doing a nested require([]) call for the build layer that has the shimmed script.
* local, 복수 파일 빌드, CDN이 지원됩니다. 어떤 shim된 스크립트가 있다면, 그의 디펜던시는 shim된 스크립트 실행전에 로딩되어야만 합니다. 이것은 shim된 스크립트를 포함하는 빌드레이어 안에서 디펜던시를 직접적으로 빌드하거나 또는 이의 디펜던시들을 shim된 스크립트를 가지고 있는 빌드레이어 내에서 require([], function(){}) 호출로 불러온 후에 require([]) 호출을 실행해야합니다.
* If you are using uglifyjs to minify the code, **do not** set the uglify option toplevel to true, or if using the command line **do not** pass -mt. That option mangles the global names that shim uses to find exports.
* 만약 당신이 ugilifyjs를 사용해서 코드를 압축한다면, uglify 옵션을 toplevel을 true로 놓지 말거나 커맨드 라인을 사용한다면 -mt를 넘겨주면 안됩니다. 이 옵션은 shim이 exports를 찾기 위해 사용하는 전역 name들을 훼손시킵니다. 

**map**: For the given module prefix, instead of loading the module with the given ID, substitute a different module ID.

**map**: 모듈 prefix가 주어져있을 때, 모듈을 주어진 ID로 부르는 대신에 다른 모듈 ID로 대체하는 것입니다. 

This sort of capability is really important for larger projects which may have two sets of modules that need to use two different versions of 'foo', but they still need to cooperate with each other.

이러한 종류의 기능은 보다 큰 프로젝트를 할 때 매우 중요합니다. 예를 들면 두 개의 다른 버전의 'foo'를 사용할 필요가 있으면서 서로 협력해야하는 두 묶음 모듈을 가지고 있는 프로젝트 같은 곳에서 말입니다.

This is not possible with the context-backed multiversion support. In addition, the paths config is only for setting up root paths for module IDs, not for mapping one module ID to another one.

context-backed multiversion support는 어렵습니다. path설정은 모듈 ID들에 대한 루트 패스 를 설정해줄 뿐입니다. 하나의 모듈ID를 다른 것에 매핑해주는 것은 아닙니다. 

map example:

map의 예제:
```javascript
requirejs.config({
    map: {
        'some/newmodule': {
            'foo': 'foo1.2'
        },
        'some/oldmodule': {
            'foo': 'foo1.0'
        }
    }
});
```
If the modules are laid out on disk like this:

모듈은 디스크 상에 다음과 같이 위치되어 있습니다:
```
foo1.0.js
foo1.2.js
some/
	newmodule.js
	oldmodule.js
```

When 'some/newmodule' does 'require('foo')' it will get the foo1.2.js file, and when 'some/oldmodule' does 'require('foo')' it will get the foo1.0.js file.

'some/newmodule'이 'require('foo')'를 하게되면 fool.2.js 파일을 가져온다. 'some/oldmodule'이 'require('foo')'를 하게되면 fool.0.js 파일을 가져올 것이다. 

This feature only works well for scripts that are real AMD modules that call define() and register as anonymous modules. Also, **only use absolute module IDs** for map config. Relative IDs (like '../some/thing') do not work.

이 기능은 오직 스크립트가 define()을 호출하고 익명 모듈을 등록하는 실제 AMD 모듈일 때만 잘 동작합니다. 또한 map 설정에 있어서 **절대 모듈ID만을 사용**합니다. 상대ID('../some/thing'과 같은)는 동작하지 않습니다. 

There is also support for a "*" map value which means "for all modules loaded, use this map config". If there is a more specific map config, that one will take precedence over the star config. Example:

"*" map 값도 지원을 합니다. 이것은 "모든 모듈이 로딩되면, 이 map 설정을 사용하여라" 와 같은 의미가 됩니다. 더 명시적인 map 설정이 있다면, 그것은 "*" 설정에 우선합니다. 예를 들어:
```javascript
requirejs.config({
    map: {
        '*': {
            'foo': 'foo1.2'
        },
        'some/oldmodule': {
            'foo': 'foo1.0'
        }
    }
});
```
Means that for any module except "some/oldmodule", when "foo" is wanted, use "foo1.2" instead. For "some/oldmodule" only, use "foo1.0" when it asks for "foo".

이것은 "some/oldmodule"을 제외한 모든 모듈에 대해 "foo"를 사용하게 되면 "foo1.2"를 사용하라는 의미가 됩니다. 오직 "some/oldmodule" 만이 "foo"를 찾찾게 되면 "foo1.0"을 사용합니다. 

**Note**: when doing builds with map config, the map config needs to be fed to the optimizer, and the build output must still contain a requirejs config call that sets up the map config. The optimizer does not do ID renaming during the build, because some dependency references in a project could depend on runtime variable state. So the optimizer does not invalidate the need for a map config after the build.

**참고**: map 설정으로 빌드를 할 때 map 설정은 optimizer에 들어가야 하며, 산출물 빌드는 map 설정을 준비하는 RequireJS 설정 호출을 반드시 포함하고 있어야 합니다. optimizer는 빌드 동안에 ID명을 변경하지 않습니다. 프로젝트 내의 몇몇 디펜던시 참조가 런타임 변수 상태에 의존할 수 있기 때문입니다. 그렇기 때문에 optimizer는 빌드 후에 map 설정을 필요로 하지 않습니다. 

**config**: There is a common need to pass configuration info to a module. That configuration info is usually known as part of the application, and there needs to be a way to pass that down to a module. In RequireJS, that is done with the **config** option for requirejs.config(). Modules can then read that info by asking for the special dependency "module" and calling **module.config()**. Example:

**config**: 설정정보를 모듈안으로 넘기려는 일반적인 필요성이 있었습니다. 설정 정보는 보통 애플리케이션의 일부로 알려져 있고, 그것들은 모듈 안으로 넘길 방법이 필요했습니다. RequireJS에서는, requirejs.config()의 **config** 옵션을 통해 사용할 수 있습니다. 모듈들은 이 정보를 특정 디펜던시 "모듈"을 요청하고 **module.confing()**를 호출 함으로써 정보를 읽을 수 있게 됩니다. 예제는 다음과 같습니다 : 
```javascript
requirejs.config({
    config: {
        'bar': {
            size: 'large'
        },
        'baz': {
            color: 'blue'
        }
    }
});

//bar.js, which uses simplified CJS wrapping:
//http://requirejs.org/docs/whyamd.html#sugar
define(function (require, exports, module) {
    //Will be the value 'large'
    var size = module.config().size;
});

//baz.js which uses a dependency array,
//it asks for the special module ID, 'module':
//https://github.com/jrburke/requirejs/wiki/Differences-between-the-simplified-CommonJS-wrapper-and-standard-AMD-define#wiki-magic
define(['module'], function (module) {
    //Will be the value 'blue'
    var color = module.config().color;
});
```
For passing config to a package, target the main module in the package, not the package ID:

config를 패키지에 넘겨주게 되면, 패키지ID가 아닌 패키지 안의 메인 모듈을 대상으로 하게됩니다:
```javascript
requirejs.config({
    //Pass an API key for use in the pixie package's
    //main module.
    config: {
        'pixie/index': {
            apiKey: 'XJKDLNS'
        }
    },
    //Set up config for the "pixie" package, whose main
    //module is the index.js file in the pixie folder.
    packages: [
        {
            name: 'pixie',
            main: 'index'
        }
    ]
});
```
**packages**: configures loading modules from CommonJS packages. See the packages topic for more information.

**package**: CommonJS 페키지에서 설정 로딩 모듈입니다. 더 많은 정보를 원하신다면 package topic을 보시기 바랍니다.

**nodeIdCompat**: Node treats module ID example.js and example the same. By default these are two different IDs in RequireJS. If you end up using modules installed from npm, then you may need to set this config value to true to avoid resolution issues.

**nodeIdCompat**: Node는 모듈ID example.js와 example을 같게 처리한다. RequireJS에서는 둘을 다른 것이 됩니다. 만약 npm에서 설치된 모듈을 사용하게 되었다면, 이 설정값을 true로 해서 resolution 이슈를 피하기 바랍니다. 

**waitSeconds**: The number of seconds to wait before giving up on loading a script. Setting it to 0 disables the timeout. The default is 7 seconds.

**waitSeconds**: 스크립트 로딩을 기다리는 시간입니다. 초단위입니다. 타임아웃되지 않도록 이것을 설정할 수 있습니다. 디폴트는 7초입니다.

**context**: A name to give to a loading context. This allows require.js to load multiple versions of modules in a page, as long as each top-level require call specifies a unique context string. To use it correctly, see the Multiversion Support section.

**context**: 로딩 context에 주어지는 이름입니다. 이것은 require.js가 top-level require 호출이 유니크한 context 문자열을 명시해놓는 동안 한 페이지 안에 다양한 버전의 모듈을 로딩하도록 해줍니다. 정확하게 사용하기 위해서 Multiversion Support 섹션을 살펴보기 바랍니다.

**deps**: An array of dependencies to load. Useful when require is defined as a config object before require.js is loaded, and you want to specify dependencies to load as soon as require() is defined. Using deps is just like doing a require([]) call, but done as soon as the loader has processed the configuration. **It does not block** any other require() calls from starting their requests for modules, it is just a way to specify some modules to load asynchronously as part of a config block.

**deps**: 로딩할 디펜던시들의 배열입니다. require가 require.js가 로딩되기 전에 config 객체로 정의될 때 유용합니다. deps는 require([]) 호출과 비슷하게 사용하면 되지만, 로더가 설정을 진행하자마자 실행됩니다. 이것은 모듈들에 대한 요청을 시작하는 다른 require() 호출에 의해 막히는 일이 없으며, 이것은 단지 설정 블락의 부분으로써 비동기적으로 로딩되는 모듈을 명시하는 방법일 뿐입니다.

**callback**: A function to execute after **deps** have been loaded. Useful when require is defined as a config object before require.js is loaded, and you want to specify a function to require after the configuration's **deps** array has been loaded.

**callback**: deps가 로딩된 후에 실행되는 함수입니다. require가 requre.js가 로딩되기 전에 설정 객체로서 정의될 때와 설정의 deps 배열이 로딩된 후에 require될 함수를 명시하고자 할 때 유용합니다.

**enforceDefine**: If set to true, an error will be thrown if a script loads that does not call define() or have a shim exports string value that can be checked. See Catching load failures in IE for more information.

**enforceDefine**: 이것이 true로 설정되면, 스크립트 로딩이 define()을 호출하지 않았거나 shim exports 문자열 값이 체크될 수 있을 때 에러가 던져집니다. Catching load failure in IE를 보시면 더 많은 정보를 얻을 수 있습니다.

**xhtml**: If set to true, document.createElementNS() will be used to create script elements.

**xhtml**: true로 설정되었다면, document.createElementNS()를 통해 스크립트 엘리먼트를 생성할 것입니다.

**urlArgs**: Extra query string arguments appended to URLs that RequireJS uses to fetch resources. Most useful to cache bust when the browser or server is not configured correctly. Example cache bust setting for urlArgs:

**urlArgs**: RequireJS 리소스를 불러오기 위해 사용하는 URL에 추가적인 쿼리 문자열 인자를 덧붙입니다. 브라우저나 서버가 올바르게 설정되지 않았을 때 캐쉬를 날리는 가장 효과적인 방법입니다. urlArgs를 사용해 캐쉬를 날리는 예제는 다음과 같습니다:
```javascript
urlArgs: "bust=" +  (new Date()).getTime()
```
During development it can be useful to use this, however **be sure** to remove it before deploying your code.

**개발하는 동안 이것을 사용하는 것이 유용할 수 있지만, 코드를 deploying 하기 전에는 제거해야합니다.**

**scriptType**: Specify the value for the type="" attribute used for script tags inserted into the document by RequireJS. Default is "text/javascript". To use Firefox's JavaScript 1.8 features, use "text/javascript;version=1.8".

**scriptType**: RequireJS를 사용해서 문서 안에 삽입될 스크립트 태그의 type="" 어트리뷰트의 값을 명시합니다. 디폴트는 "text/javascript" 입니다. 파이어폭스의 Javascript 1.8의 기능을 이용하려면, "text/javascipt;version=1.8"이라고 작성하면 됩니다.

**skipDataMain**: Introduced in RequireJS 2.1.9: If set to true, skips the data-main attribute scanning done to start module loading. Useful if RequireJS is embedded in a utility library that may interact with other RequireJS library on the page, and the embedded version should not do data-main loading.

**skipDataMain**: RequireJS 2.1.9부터 사용가능합니다 : 이것이 true로 설정되어 있다면 data-main 어트리뷰트의 스케닝을 스킵하고 모듈 로딩을 시작합니다. RequireJS가 페이지 상의 RequireJS 라이브러리와 상호작용할 유틸리티 라이브러리 안에 들어있을 때 embeded 버전이 data-main 로딩을 하지 말아야 할 때 유용합니다.

<a name="Advanced_Usage">
## ADVANCED USAGE
<a name="Loading_Modules_from_Packages">
### Loading Modules from Packages
RequireJS supports loading modules that are in a CommonJS Packages directory structure, but some additional configuration needs to be specified for it to work. Specifically, there is support for the following CommonJS Packages features:
* A package can be associated with a module name/prefix.
* The package config can specify the following properties for a specific package:
 * name: The name of the package (used for the module name/prefix mapping)
 * location: The location on disk. Locations are relative to the baseUrl configuration value, unless they contain a protocol or start with a front slash (/).
 * main: The name of the module inside the package that should be used when someone does a require for "packageName". The default value is "main", so only specify it if it differs from the default. The value is relative to the package folder.

**IMPORTANT NOTES**
* While the packages can have the CommonJS directory layout, the modules themselves should be in a module format that RequireJS can understand. Exception to the rule: if you are using the r.js Node adapter, the modules can be in the traditional CommonJS module format. You can use the CommonJS converter tool if you need to convert traditional CommonJS modules into the async module format that RequireJS uses.
* Only one version of a package can be used in a project context at a time. You can use RequireJS multiversion support to load two different module contexts, but if you want to use Package A and B in one context and they depend on different versions of Package C, then that will be a problem. This may change in the future.

If you use a similar project layout as specified in the Start Guide, the start of your web project would look something like this (Node/Rhino-based projects are similar, just use the contents of the **scripts** directory as the top-level project directory):
```
project-directory/
	project.html
	scripts/
		require.js
```
Here is how the example directory layout looks with two packages, **cart** and **store**:
```
project-directory/
	project.html
	scripts/
		cart/
			main.js
		store/
			main.js
			util.js
		main.js
		require.js
```
**project.html** will have a script tag like this:
```html
<script data-main="scripts/main" src="scripts/require.js"></script>
```
This will instruct require.js to load scripts/main.js. **main.js** uses the "packages" config to set up packages that are relative to require.js, which in this case are the source packages "cart" and "store":
```javascript
//main.js contents
//Pass a config object to require
require.config({
    "packages": ["cart", "store"]
});

require(["cart", "store", "store/util"],
function (cart,   store,   util) {
    //use the modules as usual.
});
```
A require of "cart" means that it will be loaded from **scripts/cart/main.js**, since "main" is the default main module setting supported by RequireJS. A require of "store/util" will be loaded from **scripts/store/util.js**.

If the "store" package did not follow the "main.js" convention, and looked more like this:
```
project-directory/
	project.html
	scripts/
		cart/
			main.js
		store/
			store.js
			util.js
		main.js
		package.json
		require.js
```
Then the RequireJS configuration would look like so:
```javascript
require.config({
    packages: [
        "cart",
        {
            name: "store",
            main: "store"
        }
    ]
});
```
To avoid verbosity, it is strongly suggested to always use packages that use "main" convention in their structure.

<a name="Multiversion_Support">
### Multiversion Support
As mentioned in Configuration Options, multiple versions of a module can be loaded in a page by using different "context" configuration options. require.config() returns a require function that will use the context configuration. Here is an example that loads two different versions of the alpha and beta modules (this example is taken from one of the test files):
```html
<script src="../require.js"></script>
<script>
var reqOne = require.config({
  context: "version1",
  baseUrl: "version1"
});

reqOne(["require", "alpha", "beta",],
function(require,   alpha,   beta) {
  log("alpha version is: " + alpha.version); //prints 1
  log("beta version is: " + beta.version); //prints 1

  setTimeout(function() {
    require(["omega"],
      function(omega) {
        log("version1 omega loaded with version: " +
             omega.version); //prints 1
      }
    );
  }, 100);
});

var reqTwo = require.config({
      context: "version2",
      baseUrl: "version2"
    });

reqTwo(["require", "alpha", "beta"],
function(require,   alpha,   beta) {
  log("alpha version is: " + alpha.version); //prints 2
  log("beta version is: " + beta.version); //prints 2

  setTimeout(function() {
    require(["omega"],
      function(omega) {
        log("version2 omega loaded with version: " +
            omega.version); //prints 2
      }
    );
  }, 100);
});
</script>
```
Note that "require" is specified as a dependency for the module. This allows the require() function that is passed to the function callback to use the right context to load the modules correctly for multiversion support. If "require" is not specified as a dependency, then there will likely be an error.

<a name="Loading_Code_After_Page_Load">
### Loading Code After Page Load
The example above in the Multiversion Support section shows how code can later be loaded by nested require() calls.

<a name="Web_Worker_Support">
### Web Worker Support
As of release 0.12, RequireJS can be run inside a Web Worker. Just use importScripts() inside a web worker to load require.js (or the JS file that contains the require() definition), then call require.

You will likely need to set the **baseUrl** configuration option to make sure require() can find the scripts to load.

You can see an example of its use by looking at one of the files used in the unit test.

<a name="Rhino_Support">
### Rhino Support
RequireJS can be used in Rhino via the r.js adapter. See the r.js README for more information.

<a name="Handling_Errors">
### Handling Errors
The general class of errors are 404s for scripts (not found), network timeouts or errors in the scripts that are loaded. RequireJS has a few tools to deal with them: require-specific errbacks, a "paths" array config, and a global requirejs.onError.

The error object passed to errbacks and the global requirejs.onError function will usually contain two custom properties:
* **requireType**: A string value with a general classification, like "timeout", "nodefine", "scripterror".
* **requireModules**: an array of module names/URLs that timed out.

If you get an error with a requireModules, it probably means other modules that depend on the modules in that requireModules array are not defined.

<a name="Catching_load_failures_in_IE">
#### Catching load failures in IE
Internet Explorer has a set of problems that make it difficult to detect load failures for errbacks/paths fallbacks:
* script.onerror does not work in IE 6-8. There is no way to know if loading a script generates a 404, worse, it triggers the onreadystatechange with a complete state even in a 404 case.
* script.onerror does work in IE 9+, but it has a bug where it does not fire script.onload event handlers right after execution of script, so it cannot support the standard method of allowing anonymous AMD modules. So script.onreadystatechange is still used. However, onreadystatechange fires with a complete state before the script.onerror function fires.

So it is very difficult with IE to allow both anonymous AMD modules, which are a core benefit of AMD modules, and reliable detect errors.

However, if you are in a project that you know uses define() to declare all of its modules, or it uses the shim config to specify string exports for anything that does not use define(), then if you set the enforceDefine config value to true, the loader can confirm if a script load by checking for the define() call or the existence of the shim's exports global value.

So if you want to support Internet Explorer, catch load errors, and have modular code either through direct define() calls or shim config, always set **enforceDefine** to be true. See the next section for an example.

**NOTE**: If you do set enforceDefine: true, and you use data-main="" to load your main JS module, then that main JS module **must call define()** instead of require() to load the code it needs. The main JS module can still call require/requirejs to set config values, but for loading modules it should use define().

If you then also use almond to build your code without require.js, be sure to use the insertRequire build setting to insert a require call for the main module -- that serves the same purpose of the initial require() call that data-main does.

<a name="require_errbacks">
#### require([]) errbacks
Errbacks, when used with requirejs.undef(), will allow you to detect if a module fails to load, undefine that module, reset the config to a another location, then try again.

A common use case for this is to use a CDN-hosted version of a library, but if that fails, switch to loading the file locally:
```javascript
requirejs.config({
    enforceDefine: true,
    paths: {
        jquery: 'http://ajax.googleapis.com/ajax/libs/jquery/1.4.4/jquery.min'
    }
});

//Later
require(['jquery'], function ($) {
    //Do something with $ here
}, function (err) {
    //The errback, error callback
    //The error has a list of modules that failed
    var failedId = err.requireModules && err.requireModules[0];
    if (failedId === 'jquery') {
        //undef is function only on the global requirejs object.
        //Use it to clear internal knowledge of jQuery. Any modules
        //that were dependent on jQuery and in the middle of loading
        //will not be loaded yet, they will wait until a valid jQuery
        //does load.
        requirejs.undef(failedId);

        //Set the path to jQuery to local path
        requirejs.config({
            paths: {
                jquery: 'local/jquery'
            }
        });

        //Try again. Note that the above require callback
        //with the "Do something with $ here" comment will
        //be called if this new attempt to load jQuery succeeds.
        require(['jquery'], function () {});
    } else {
        //Some other error. Maybe show message to the user.
    }
});
```
With `requirejs.undef()`, if you later set up a different config and try to load the same module, the loader will still remember which modules needed that dependency and finish loading them when the newly configured module loads.

**Note**: errbacks only work with callback-style require calls, not define() calls. define() is only for declaring modules.

<a name="paths_config_fallbacks">
#### paths config fallbacks
The above pattern for detecting a load failure, undef()ing a module, modifying paths and reloading is a common enough request that there is also a shorthand for it. The paths config allows array values:
```javascript
requirejs.config({
    //To get timely, correct error triggers in IE, force a define/shim exports check.
    enforceDefine: true,
    paths: {
        jquery: [
            'http://ajax.googleapis.com/ajax/libs/jquery/1.4.4/jquery.min',
            //If the CDN location fails, load from this location
            'lib/jquery'
        ]
    }
});

//Later
require(['jquery'], function ($) {
});
```
This above code will try the CDN location, but if that fails, fall back to the local lib/jquery.js location.

**Note**: paths fallbacks only work for exact module ID matches. This is different from normal paths config which can apply to any part of a module ID prefix segment. Fallbacks are targeted more for unusual error recovery, not a generic path search path solution, since those are inefficient in the browser.

<a name="Global_requirejs_onError_function">
#### Global requirejs.onError function
To detect errors that are not caught by local errbacks, you can override requirejs.onError():
```javascript
requirejs.onError = function (err) {
    console.log(err.requireType);
    if (err.requireType === 'timeout') {
        console.log('modules: ' + err.requireModules);
    }

    throw err;
};
```

<a name="Loader_Plugins">
## LOADER PLUGINS
RequireJS supports loader plugins. This is a way to support dependencies that are not plain JS files, but are still important for a script to have loaded before it can do its work. The RequireJS wiki has a list of plugins. This section talks about some specific plugins that are maintained alongside RequireJS:

<a name="Specify_a_Text_File_Dependency">
### Specify a Text File Dependency
It is nice to build HTML using regular HTML tags, instead of building up DOM structures in script. However, there is no good way to embed HTML in a JavaScript file. The best that can be done is using a string of HTML, but that can be hard to manage, particularly for multi-line HTML.

RequireJS has a plugin, text.js, that can help with this issue. It will automatically be loaded if the text! prefix is used for a dependency. See the [text.js README](https://github.com/requirejs/text) for more information.

<a name="Page_Load_Event_Support_DOM_Ready">
### Page Load Event Support/DOM Ready
It is possible when using RequireJS to load scripts quickly enough that they complete before the DOM is ready. Any work that tries to interact with the DOM should wait for the DOM to be ready. For modern browsers, this is done by waiting for the DOMContentLoaded event.

However, not all browsers in use support DOMContentLoaded. The domReady module implements a cross-browser method to determine when the DOM is ready. Download the module and use it in your project like so:
```javascript
require(['domReady'], function (domReady) {
  domReady(function () {
    //This function is called once the DOM is ready.
    //It will be safe to query the DOM and manipulate
    //DOM nodes in this function.
  });
});
```
Since DOM ready is a common application need, ideally the nested functions in the API above could be avoided. The domReady module also implements the Loader Plugin API, so you can use the loader plugin syntax (notice the ! in the domReady dependency) to force the require() callback function to wait for the DOM to be ready before executing. domReady will return the current document when used as a loader plugin:
```javascript
require(['domReady!'], function (doc) {
    //This function is called once the DOM is ready,
    //notice the value for 'domReady!' is the current
    //document.
});
```
**Note**: If the document takes a while to load (maybe it is a very large document, or has HTML script tags loading large JS files that block DOM completion until they are done), using domReady as a loader plugin may result in a RequireJS "timeout" error. If this a problem either increase the waitSeconds configuration, or just use domReady as a module and call domReady() inside the require() callback.

<a name="Define_an_I18N_Bundle">
### Define an I18N Bundle(I18N 번들 정의하기)
Once your web app gets to a certain size and popularity, localizing the strings in the interface and providing other locale-specific information becomes more useful. However, it can be cumbersome to work out a scheme that scales well for supporting multiple locales.

웹 응용 프로그램은 특정 크기와 인기에 도달하면, 인터페이스에서 문자열을 지역화하고 다른 지역별 정보를 제공하는 것이 더 유용하게 됩니다. 그렇지만, 여러 지역을 지원하기 위해 잘 조절하는 구조를 만드는 것은 부담이 될 수 있습니다. 

RequireJS allows you to set up a basic module that has localized information without forcing you to provide all locale-specific information up front. It can be added over time, and only strings/values that change between locales can be defined in the locale-specific file.

RequireJS는 당신이 앞에 모든 지역별 정보를 제공하지 않고 정보를 지역화 한 적이있는 기본 모듈을 설정할 수 있습니다. 그것은 시간이 지나도 추가할 수 있고, 지역간에 변하는 문자열/값을 지역별 파일에 정의할 수 있습니다.

i18n bundle support is provided by the i18n.js plugin. It is automatically loaded when a module or dependency specifies the i18n! prefix (more info below). [Download the plugin](http://requirejs.org/docs/download.html#i18n) and put it in the same directory as your app's main JS file.

i18n 번들 제공은 i18n.js 플러그인에 의해 제공됩니다. 모듈이나 디펜던시가 i18n! 접두사로 지정할 때 자동으로 로딩됩니다. 플러그인을 다운로드하고 앱의 main JS 파일과 같은 디렉토리에 넣어주십시오.

To define a bundle, put it in a directory called "nls" -- the i18n! plugin assumes a module name with "nls" in it indicates an i18n bundle. The "nls" marker in the name tells the i18n plugin where to expect the locale directories (they should be immediate children of the nls directory). If you wanted to provide a bundle of color names in your "my" set of modules, create the directory structure like so:

번들을 정의하기 위해 nls 디렉토리를 생성하십시오. i18n! 플러그인은 i18n 번들이 가리키는 것으로 "nls"로 모듈 이름을 지정합니다. "nls" 이름은 i18n 플러그인이 찾는 로케일 디렉토리입니다(nls 디렉토리 아래 디렉토리에 있어야합니다.). 당신이 모듈의 "my"에서 색 이름의 번들을 제공하기 원한다면, 아래와 같이 디렉토리 구성을 생성해야 합니다:
* my/nls/colors.js

The contents of that file should look like so:

파일은 아래와 같습니다:
```javascript
//my/nls/colors.js contents:
define({
    "root": {
        "red": "red",
        "blue": "blue",
        "green": "green"
    }
});
```
An object literal with a property of "root" defines this module. That is all you have to do to set the stage for later localization work.

이 모듈은 "root" 속성과 함께 객체 리터럴을 정의하였다. 저것이 당신이 나중에 현지화 작업을 위한 단계에서 설정하는 모든 것입니다.

You can then use the above module in another module, say, in a my/lamps.js file:

당신은 다른 모듈에서 위 모듈을 사용할 수 있습니다:
```javascript
//Contents of my/lamps.js
define(["i18n!my/nls/colors"], function(colors) {
    return {
        testMessage: "The name for red in this locale is: " + colors.red
    }
});
```
The my/lamps module has one property called "testMessage" that uses colors.red to show the localized value for the color red.

my/lamps 모듈은 "testMessage" 라로 불리는 하나의 속성을 가지고 있고 이것은 빨간색을 위한 지역화 값을 보여주기 위해 colors.red를 사용합니다.

Later, when you want to add a specific translation to a file, say for the fr-fr locale, change my/nls/colors to look like so:

나중에, 파일에 특정 번역을 추가하고 싶을 때, fr-fr 지역을 말하자면, my/nls/colors 를 아래와 같이 변경합니다:
```javascript
//Contents of my/nls/colors.js
define({
    "root": {
        "red": "red",
        "blue": "blue",
        "green": "green"
    },
    "fr-fr": true
});
```
Then define a file at my/nls/fr-fr/colors.js that has the following contents:

그리고 my/nls/fr-fr/colors.js 파일을 아래와 같이 정의하십시오:
```javascript
//Contents of my/nls/fr-fr/colors.js
define({
    "red": "rouge",
    "blue": "bleu",
    "green": "vert"
});
```
RequireJS will use the browser's navigator.language or navigator.userLanguage property to determine what locale values to use for my/nls/colors, so your app does not have to change. If you prefer to set the locale, you can use the module config to pass the locale to the plugin:

RequireJS는 my/nls/colors를 사용하기위한 로케일 값을 결정하기 위해 브라우저의 navigator.language나 navigator.userLanguage 속성을 사용한다. 그리고 애플리케이션의 바꾸지 않는다. 만약 당신이 로케일을 설정하기를 원한다면, 당신은 플러그인에 로케일을 전달하는 모듈 설정을 사용할 수 있습니다:
```javascript
requirejs.config({
    config: {
        //Set the config for the i18n
        //module ID
        i18n: {
            locale: 'fr-fr'
        }
    }
});
```
**Note** that RequireJS will always use a lowercase version of the locale, to avoid case issues, so all of the directories and files on disk for i18n bundles should use lowercase locales.

**주의사항** RequireJS는 대소문자 이슈를 피하기 위해 로케일 버전을 소문자로 사용합니다. 그래서 i18n을 위한 모든 디렉토리나 파일은 꼭 소문자를 사용해야만 합니다.

RequireJS is also smart enough to pick the right locale bundle, the one that most closely matches the ones provided by my/nls/colors. For instance, if the locale is "en-us", then the "root" bundle will be used. If the locale is "fr-fr-paris" then the "fr-fr" bundle will be used.

RequireJS는 또한 my/nls/colors에서 제공하는 것중에 가장 가까운 것 하나를 선택 할 정도로 똑똑합니다. 예를 들어서 "en-us" 로케일이면 "root" 번들을 사용할 것이다. 만약 "fr-fr-paris" 라면 "fr-fr" 번들을 사용할 것입니다.

RequireJS also combines bundles together, so for instance, if the french bundle was defined like so (omitting a value for red):

또한 RequireJS는 아래와 같이 french 번들이 빨간색이 생략되어 아래와 같이 정의 되어 있을 경우 번들들을 조합합니다.:
```javascript
//Contents of my/nls/fr-fr/colors.js
define({
    "blue": "bleu",
    "green": "vert"
});
```
Then the value for red in "root" will be used. This works for all locale pieces. If all the bundles listed below were defined, then RequireJS will use the values in the following priority order (the one at the top takes the most precedence):

이때 "root"에 있는 red 값이 사용됩니다. 이것은 모든 로케일 조각에서 작동합니다. 만약 아래와 같이 모든 번들이 정의되었다면, RequireJS는 상단에 있는 것을 가장 우선으로 해서 아래의 순서대로 값을 사용할 것입니다.
* my/nls/fr-fr-paris/colors.js
* my/nls/fr-fr/colors.js
* my/nls/fr/colors.js
* my/nls/colors.js

If you prefer to not include the root bundle in the top level module, you can define it like a normal locale bundle. In that case, the top level module would look like:

만약 상위 모듈에서 root 번들이 포함되는 것을 원하지 않으면, 기본 로케일 번들처럼 root 번들을 정의할 수 있다. 이 경우, 상위 모듈은 아래와 같다:
```javascript
//my/nls/colors.js contents:
define({
    "root": true,
    "fr-fr": true,
    "fr-fr-paris": true
});
```
and the root bundle would look like:
그리고 root 번들은 아래와 같다:
```javascript
//Contents of my/nls/root/colors.js
define({
    "red": "red",
    "blue": "blue",
    "green": "green"
});
```

Latest Release: 2.1.14  
Open source: new BSD or MIT licensed  
web design by Andy Chung © 2011-2014  

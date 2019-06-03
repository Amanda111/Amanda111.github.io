title: JS Modularity
date: 2016-03-17 12:36:20
tags: 
---

## what is modularity？
> JavaScript modules are encapsulated, meaning that they keep implementation details private, and expose a public API.

**modules should be**

> ***specialized:***  一个模块是专门解决一个问题的

> ***independent:***  每个模块是独立的。模块之间由API连接

> ***decomposable:*** 可分解

> ***recomposable:*** 可组合

> ***substitutable:*** 可被代替

-------

##  why modularity？
*Code complexity grows. And modularity simplifies things!*

**solution to name collision**

当一个项目大起来的时候，js的代码也越来越多，不进行闭包或者模块化就会产生大量的全局变量，而造成了命名冲突。

**safely  make change**

在我们写一个app的时候，整个app可能由几个部分构成，且它们之间相互联系。在我们没有使每个部分模块化的情况下，对它们进行修改的时候还要考虑到它们对整体的影响。所以我们在进行修改之前，还要go through the whole program, 真是件吃力的事情。但如果把它们都模块化，就可以在这个模块里面直接修改，而不用担心它会对整个项目产生bad effect.

同样，也不用担心修改其他的部分会影响到它.

**reuse the modules**

模块可以被重复使用，and its API should be clean

**make the code more readable**

也使代码的结构性更强。

-------------------------

### function
*只能实现简单的封装。*

	var math = {
		add: function add(a,b){
			return a + b
		},
		sub: function sub(a,b){
			return a - b
		}
	}
	console.log(math.add(1,2));
	console.log(add(1,2));//add is undefined

-----------------------

### IIFE: Immediately-invoked Function Expression
*能实现访问控制。*

	var module =(function(){
		var private = "private";
		var foo = function(){
			console.log(private);
		}
		return{
			foo:foo
		}
	})();
	console.log(module.foo());
	console.log(module.private);//undefined

--------------

function和IIFE都没有依赖声明。

**我们该怎么更好地模块化呢? **

## CommonJS & AMD & ES6 

-------

### CommonJS
**多用于server上。不适合浏览器使用，不能发送异步请求。同步require**

	//content of foo.js 
	//先定义一个foo模块
	var foo = function () {
	  return 'foo method result';
	};
	//再把foo模块暴露给其他模块
	exports.method = foo

	//content of bar.js
	var Foo = require('../foo');//声明依赖foo模块
	var barMethod = function () {
	  return 'barMethod result';
	};
	var fooMethod = function () {
	  return Foo.method();
	};
	//export 
	exports.barMethod = barMethod;
	exports.fooMethod = fooMethod;

	//依赖bar.js 这个模块
	var bar = require('bar');
	bar.barMethod();
	bar.fooMethod();

----------
### AMD (Asynchronous Module Definition)

AMD里面模块是被**异步加载**的，加载完后就在缓存里。这能很好地**适应浏览器的环境**，因为这就不需要在每次加载应用时候，把所有的模块都重新加载一遍。而Commonjs，并不能做到这一点，所以说它不适合浏览器环境。

因为AMD要require一个define过的模块，要立刻获取define模块的内容，所以可以看到**define{}里面的内容都要用return{}包起来**

AMD还可以加载html，css文件

It should look like this

	// content of foo.js  define('module's name',function(){}) 定义模块foo
	define('foo',function(){
		return{
			method: function(){
				return "foo method"
			}
		}
	});

	//content of bar.js  define('module's name',['dependency'],function(){}) 定义依赖模块foo的模块bar
	define('bar',['foo'],function(){
		return{
			barMethod{
				return "bar method"
			}
			fooMethod{
				return foo.method();
			}
		}
	});

	//require(['dependency'],function(){}) 获取模块
	require(['bar'],function(){
		bar.barMethod();
	  	bar.fooMethod();
	})


--------------

### ES6

#### HERE come the export and import keywords!

**export**

	export var color = "red";
	export let name = "Nicholas";
	export const magicNumber = 7;

	export function sum(num1, num2) {
	    return num1 + num2;
	}

	//export later
	function multiply(num1, num2) {
	    return num1 * num2;
	}
	export multiply;

**import{}**

	import {identifier1,identifier2,..} from "file"

	//import everything
	import * from "file"




#### some rules we need to know

+ 一切都"use strict"

+ 只有被export了，才能在其他模块被引用。

+ module的顶级作用域不能用this这个语句

+ 需要给每个function和class起名字。但default可以有

```
	//export  default
	export default function(num1, num2) {
    	return num1 + num2;
	}
	import sum from "example"
	console.log(sum(1,2));//3
```

+ 可以rename

```
	function sum(num1, num2) {
	    return num1 + num2;
	}
	export { sum as add };//在export里改
	import { sum as add } from "example"; //在import里改will do

	console.log(add(1,2));//3
	console.log(typeof sum)//undefined
```
***References***

[1]<a href="https://leanpub.com/understandinges6/read/#leanpub-auto-modules">Understanding ECMAScript6-Modules</a>

[2]<a href="https://www.safaribooksonline.com/library/view/programming-javascript-applications/9781491950289/ch04.html">Programming JavaScript Applications-Chapter 4. Modules</a>

[3]<a href="https://www.safaribooksonline.com/library/view/eloquent-javascript-2nd/9781457189821/ch10.html">Eloquent JavaScript Chapter10 Modules</a>

[4]<a href="https://github.com/vasanthk/js-bits/blob/master/js/amd-commonjs-es6modules.js">js-bits</a>

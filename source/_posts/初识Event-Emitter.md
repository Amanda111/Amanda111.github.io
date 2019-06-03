---
title: 初识Event Emitter
date: 2016-12-10 11:55:34
tags:
---

**Event Emitter** 是“观察者模式（Observer Pattern”在前端的一种呈现方式。

所谓**观察者模式**(Observer Pattern)就是定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。

比如，在知乎上，我们关注了某个用户，当这个用户有了动态，我们的TimeLine上面就会显示。如果取消关注之后，就不会再有其动态的推送了。这里，如果把该用户的动态看作是事件的话，我们就是"订阅者"，"监听"着该用户的动态;该用户就是"发布者"，他更新动态就“触发”了该事件。

回到Event emitter。之前在用Vue2.0写一个小东西，需要实现非父子组件通信的时候，发现已经没有了`$dispatch`和 `$broadcast`, 而要用`$emit`, Vue 里面的`$emit` 是什么样的呢？

## Vue $emit

### Vue文档介绍的用法：

```
//在简单的场景下，使用一个空的 Vue 实例作为中央事件总线：
var bus = new Vue()
// 触发组件 A 中的事件
bus.$emit('id-selected', 1)
// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
})

```

尝试了一下，跟着写了一个todo。部分组件结构：Todos -> todo -> delete，现在使delete组件与Todos组件这两个非父子组件用emitter通信。

**首先，创建一个新的Vue实例bus**

```
//bus.js
import Vue from 'vue'
export var bus = new Vue()

```

**然后，在存在“发布者”和“订阅者”的组件里面**`import { bus } from 'route to../bus.js'`

```
//Todos.vue
import { bus } from '../bus.js'
	export default{
		//data
		//components
		......
		created() {
		  	......
		  	bus.$on('remove',this.rmTodo)
		},
		methods:{
			......
  			rmTodo(todo){
  				this.todos.splice(this.todos.indexOf(todo),1)
  			}
		}
	}

```

```
//delete.vue
import { bus } from '../bus.js'
	export default{
		//Array todos props from parent components
		methods:{
			remove(todo){
				bus.$emit('remove',this.todo)
			}
		}
	}
```

这看上去就像，当我们触发了delete组件里的remove 方法时，该组件就会emit 一个“remove”的事件名。而监听了“remove”事件的Todos组件就能接受传入的todo参数，并执行rmTodo。

### 但实际上Event Emitter是怎样的呢？[戳开看Vue源码](https://github.com/vuejs/vue/blob/dev/src/core/instance/events.js)

主要看 `$emit` 和 `$on` 的部分

```
  Vue.prototype.$emit = function (event: string): Component {
    const vm: Component = this
    let cbs = vm._events[event]
    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs
      const args = toArray(arguments, 1)
      for (let i = 0, l = cbs.length; i < l; i++) {
        cbs[i].apply(vm, args)
      }
    }
    return vm
  }
  
  Vue.prototype.$on = function (event: string, fn: Function): Component {
    const vm: Component = this
    ;(vm._events[event] || (vm._events[event] = [])).push(fn)
    return vm
  }
```

可以看到，vm._events 就像是一个事件管理器，它存放事件名(event:string)作为key，其对应的value就是监听了该事件，并当该事件触发之后执行的所有函数

因为一个事件可能会被多个监听，所以函数存放在数组里，每有一个`$on`,就会把其函数push到对应event的value中

`(vm._events[event] || (vm._events[event] = [])).push(fn)`

当事件被触发后，`$emit`就会找到`vm._events[event]`中的函数数组，传入参数并执行回调。

知道了原理之后，接下来我们尝试...

## 实现一个简单的Event emitter

```
function Emitter(){
	//事件管理器
  this.events = {};

  this.emit = (eventName,data) =>{
    const event = this.events[eventName];
    if( event ) {
    	//遍历this.events[eventName]数组中的每一个函数，传入data并立即执行
      event.forEach(fn => {
        fn.call(null, data);
      });
    }    
  }

  this.subscribe = (eventName, fn) =>{
  	//如果this.events中没有该eventName，则创建一个
    if(!this.events[eventName]) {
      this.events[eventName] = [];
    } 
    //把fn推到数组中   
    this.events[eventName].push(fn);
  }
  
  this.off = (eventName, fn) =>{
    return () => {
    	//移除该事件里面的fn
      this.events[eventName] = this.events[eventName].filter(eventFn => fn != eventFn)
      console.log(this.events)
    }
  }
}
```

以上就是emitter构造函数的所有代码。接下来我们用它来操作一些DOM，来看看具体是怎么样用的

```
//html
<body>
 <input type="text">
 <h1></h1>
 <button id="btno">Change name</button>
 <button id="btnt">change color</button>
</body>
```

```
document.addEventListener("DOMContentLoaded", function() {
  let input = document.querySelector('input[type="text"]');
  let button = document.getElementById('btno');
  let buttont = document.getElementById('btnt')
  let h1 = document.querySelector('h1');

  //构造emitter
  let emitter = new Emitter();
  
  //绑定emit，点击该按钮时会触发‘name-changed’事件
  button.addEventListener('click', () => {
    emitter.emit('name-changed', {name: input.value});
  });
  
  //绑定了一个emit 和 off，点击该按钮时会触发‘color’事件，并取消'change'事件中的，有回调函数changeName的订阅者的订阅
  buttont.addEventListener('click', () => {
    emitter.emit('color', {color: 'red'});
    emitter.off('change',changeName)()
  });

  //定义两个function,作为订阅者的回调函数
  function changeColor(data){
    h1.style.color = data.color;
  }

  function changeName(data){
    h1.innerHTML = `Your name is: ${data.name}`;
  }
  
  //订阅者
  emitter.subscribe('change', changeName);
  emitter.subscribe('color', changeColor);
});
```
首先构造一个emitter，emitter里面有事件管理器`this.events={}`和emit，subscribe两个方法。我们试着在两个button上绑定了click用来触发emit。

再写两个订阅者，分别订阅的是“change”和“color”事件。因为之前events管理器中还没有这两个事件，所以`this.events[eventName] = []`来创建以该事件名为key，空数组作为其value的对象，并把订阅者的函数push到数组中。

当点击按钮的时候，就会向事件名对应的函数数组中的每一个函数传入data参数，执行回调

[Demo代码](https://github.com/Amanda111/event-emitter)







# Vue and Laravel

## Get Started

```html
<!DOCTYPE html>
<html>
<head>
  <title>My first Vue app</title>
  <script src="https://unpkg.com/vue"></script>
</head>
<body>
  <div id="app">
    {{ message }}
  </div>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue!'
      }
    })
  </script>
</body>
</html>
```

```html
<a v-bind:href="url"> ... </a>
<a :href="url"> ... </a>
<a v-on:click="doSomething"> ... </a>
<a @click="doSomething"> ... </a>
<a v-on:click.stop="doThis"></a> <!-- the click event's propagation will be stopped -->
<form v-on:submit.prevent="onSubmit"></form> <!-- the submit event will no longer reload the page -->
<a v-on:click.stop.prevent="doThat"></a> <!-- modifiers can be chained -->
<form v-on:submit.prevent></form> <!-- just the modifier -->
<div v-on:click.self="doThat">...</div> <!-- only trigger handler if event.target is the element itself i.e. not from a child element -->

<input @keyup.enter="submit"> <!-- Others: .enter, .tab, .delete (“Delete” and “Backspace”), .esc, .space, .up, .down, .left, .right -->
<div @click.ctrl="doSomething">Do something</div> <!-- mouse button: .left, .right, .middle -->
<input @keyup.alt.67="clear">
<button @click.ctrl="onClick">A</button>


<div class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>
<div v-bind:class="[activeClass, errorClass]"></div>
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<div v-bind:style="[baseStyles, overridingStyles]"></div>


<script>
var vm = new Vue({
  beforeCreate: function() {},
  created: function () {}, 
  beforeMount: function() {},
  mounted: function() {},
  beforeUpdate: function() {}, // after data changed.
  updated: function() {},
  beforeDestroy: function() {}, // after vm.$destroy is called
  destroy: function() {},
})
vm.a === data.a

var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: { // calculate only after data is changed. 
    reversedMessage: function () { 
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    },
    fullName: {
      // getter
      get: function () {
        return this.firstName + ' ' + this.lastName
      },
      // setter
      set: function (newValue) {
        var names = newValue.split(' ')
        this.firstName = names[0]
        this.lastName = names[names.length - 1]
      }
    }
  },
  watch: {
    // whenever data.question changes, this function will run
    // use this when you need more generic way of handling data.
    question: function (newQuestion) {
      // do something.
    }
  },
  methods: { // calculate every time it is called
    reverseMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
</script>
```

## Conditional

```html
<!-- if -->
<div id="app-3">
  <span v-if="seen">Now you see me</span>
  <div v-else-if="type === 'B'">
    B
  </div>
  <div v-else>
    Now you don't
  </div>
</div>
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input"> <!-- "key" says "don't reuse this!" -->
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
v-if: high toggle
v-show: high initial render. cannot use <template>


<!-- loop -->
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
  <ol>
    <li v-for="(item, index) in items">
      {{ parentMessage }} - {{ index }} - {{ item.message }}
    </li>
  </ol>
  <div v-for="(value, key, index) in object">
    {{ index }}. {{ key }}: {{ value }}
  </div>
  <ol>
    <template v-for="item in items">
      <li>{{ item.msg }}</li>
      <li class="divider"></li>
    </template>
  </ol>
  <ol>
    <li v-for="todo in todos" v-if="!todo.isComplete">
      {{ todo }}
    </li>
  </ol>
</div>

<script>
// Detecting Array/Object change
vm.items[indexOfItem] = newValue // ==>
Vue.set(example1.items, indexOfItem, newValue)
example1.items.splice(indexOfItem, 1, newValue)

// Dynamically adding reactive element
Vue.set(vm.userProfile, 'age', 27)
this.$set(this.userProfile, 'age', 27)
this.userProfile = Object.assign({}, this.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
</script>
```

## Handling User Input

```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>
<script>
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
</script>
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
<script>
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
</script>
```

## Component

### Basics

```html
<div id="app-7">
  <ol>
    <!--
      Now we provide each todo-item with the todo object
      it's representing, so that its content can be dynamic.
      We also need to provide each component with a "key",
      which will be explained later.
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id">
    </todo-item>
  </ol>
</div>
<script>
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: 'Vegetables' },
      { id: 1, text: 'Cheese' },
      { id: 2, text: 'Whatever else humans are supposed to eat' }
    ]
  }
})
</script>
```

### Single file Component

```html
<template>
    <p>{{ greeting }} world!</p>
</template>

<script>
module.exports = {
    data: function () {
        return {
            greeting: "hi",
        }
    }
}
</script>

<style>
p {
    font-size: 2rem;
}
</style>
```
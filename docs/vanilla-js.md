# Vanilla JavaScript

Sometimes, it's an overkill to use jQuery. 

I've covered the simple comparison between jQuery and Vanilla JavaScript. 

## Select

```js
// jQuery
var el = $('.selector');

// JavaScript
var idEl = document.getElementById('id');
var span = document.getElementsByTagName('span');
var clEl = document.getElementsByClass('abc');
var els = document.querySelectorAll("span.a, span.c");
```

## Add Event Listener

```js
var els = document.getElementsByClass('class');

for(var i = 0; i < els.length; i++) {
    el.addEventListener('click', function(event){
        event.preventDefault();
    })
}
```

[Valid events list](https://developer.mozilla.org/en-US/docs/Web/Events)
[Why you cannot use `map`, `forEach` here](https://css-tricks.com/snippets/javascript/loop-queryselectorall-matches/)
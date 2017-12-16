# Vanilla JavaScript

Sometimes, it's an overkill to use jQuery. 

I've covered the simple comparison between jQuery and Vanilla JavaScript. 

## Select

```js
// find element
var el = $('.selector');
var idEl = document.getElementById('id');
var span = document.getElementsByTagName('span');
var clEl = document.getElementsByClass('abc');
var els = document.querySelectorAll("span.a, span.c");

// Closest 
var elp = $('.selector').closest('.parent');
var idEl = document.getElementById('id').closest('.parent'); // It doesn't work in IE. 
```

[closest polyfill](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest)

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


## Add/Remove class

```js
$('.selector').addClass('no-dot-in-front');
document.getElementById('id').className += " no-dot-here-too";
document.getElementById('id').classList.add('class-name'); // It only works from IE 11 and beyond. 

$('.selector').removeClass('no-dot-in-front');
var className = document.getElemntById('id').className;
className = className.replace('class-name', '');
document.getElementById('id').classList.remove('class-name'); // It only works from IE 11 and beyond.
```

## String

```js
var steNew = str.replace('from', 'to');
var index = "abc def ghi".indexOf('def'); // -1 when not found.
```
---
layout: post
title:  "event.preventDefault() vs event.stopPropagation()"
date:   2020-03-05
excerpt: ""
category: JavaScript
tag:
- JavaScript
- event
- article
comments: true
---

## What's the difference?
`event.preventDefault()` - Only the default action will be prevented but the propagation still fires.

`event.stopPropagation()` - It prevents the event from propagating the DOM. Nothing to do with the default action. 

## event.preventDefault
The `event.preventDefault` prevents the default action that belongs to the event.
Not all events can be prevented, it depends on the property `cancelable`.

### Example

html
~~~html
<a onClick={clickWithoutPreventDefault} target="_blank" href="https://doreentseng.github.com">
    Click without event.preventDefault()
</a>
<a onClick={clickWithPreventDefault} target="_blank" href="https://doreentseng.github.com">
    Click with event.preventDefault()
</a>
~~~

js
~~~js
const clickWithoutPreventDefault = (event) => {
    alert("Link clicked, go to the specified page");
}

const clickWithPreventDefault = (event) => {
    console.log(event.cancelable); // True, the event can be prevented
    event.preventDefault();
    alert("Link clicked but the default action is prevented, page won't change");
}
~~~
demo
<iframe
    src="https://codesandbox.io/embed/eventpreventdefault-3whqn?fontsize=14&hidenavigation=1&theme=dark"
    style="width:100%; height:300px; border:0; border-radius: 4px; overflow:hidden;"
    title="event.preventDefault()"
    allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
    sandbox="allow-autoplay allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

## event.stopPropagation
The `event.stopPropagation` prevents propagation of the same event from being called.
Propagation means bubbling up to parent elements or capturing down to child elements.

The elements below are neseted. When `a` is clicked, `li` and `ul` are clicked in the meantime because of the event bubbling.

### Example
html
~~~html
<ul onClick={clickUL}>
    <li onClick={clickLI}>
        <a onClick={clickAWithoutStopPropagation} href="#">
            Click without event.stopPropagation(), a, li, ul clicked
        </a>
        <a onClick={clickAWithStopPropagation} href="#">
            Click with event.stopPropagation()
        </a>
    </li>
</ul>
~~~
js
~~~js
const clickUL = (event) => {
    alert("ul clicked")
}

const clickLI = (event) => {
    alert("li clicked")
}

const clickAWithoutStopPropagation = (event) => {
    alert("a clicked")
}

const clickAWithStopPropagation = (event) => {
    event.stopPropagation() // It stops the event from bubbling up so li and ul never fire
    alert("a clicked only")
}
~~~
demo
<iframe
    src="https://codesandbox.io/embed/blissful-blackburn-wn24y?fontsize=14&hidenavigation=1&theme=dark"
    style="width:100%; height:300px; border:0; border-radius: 4px; overflow:hidden;"
    title="event.stopPropagation()"
    allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
    sandbox="allow-autoplay allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

## References:
* [MDN: Event.preventDefault()](<https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault> "Click")
* [MDN: Event.stopPropagation()](<https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation> "Click")
* [What's the difference between event.stopPropagation and event.preventDefault?](<https://stackoverflow.com/questions/5963669/whats-the-difference-between-event-stoppropagation-and-event-preventdefault> "Click")

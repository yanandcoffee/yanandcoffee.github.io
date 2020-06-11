---
title: Accessible Modals, Part 2 - Focus Trap! Key Events!
description: In this continuation, you'll learn how to create a focus trap within your dialog modal to keep track of key events that allow keyboard users to access your perfectly designed modal.
date: 2020-06-06 03:20:00
tags:
- a11y
- keyboard
- accessibility
- modals
- focus management
- javascript
- React
- refs
---

In my [last post](https://www.yanandcoffee.com/2020/05/26/accessible-modals/), I wrote about how to create accessible modals following the W3C's [WAI-ARIA 1.1](https://www.w3.org/TR/wai-aria-practices-1.1/#dialog_modal) guidelines. Part 1 covers ARIA attributes, focus management, and some of the most common accessibility concerns that come with creating a simple dialog modal. In this part, I will be covering how to listen to events within the modal so that when you are moving inside using `Tab` / `Shift` + `Tab`, you are following the following rules:

- Users cannot interact with content outside an active dialog window, meaning they contain their own tab sequence regardless of the other outside element (there are some cases where clicking outside of the dialog will close the dialog)
- Once the dialog is open, keyboard interactions should be as follows:
  - `Tab` moves forward to the next tabbable element
    - if you are at the end, it moves to the first
  - `Shift + Tab` moves to the previous tabbable element
    - if you are at the first, it moves to the last
  - `Escape` closes the dialog


I will be walking through this [example](https://codesandbox.io/s/accessible-modals-part-2-pljct?file=/src/components/Dialog.js) to explain the implementation and I will be using the terms "modal" and "dialog" interchangably to refer to the same dialog modal UI pattern.

## What Do We Need
First, let's talk about what we need in order for us to handle the interactions based on the keys being pressed. We're going to need to listen for key events within the dialog. The most logical way to do it in React would be on the uppermost parent modal div with `onKeyDown` and a function handler, `handleKeyDown`. We don't want to listen for events on the document or window unless we absolutely have to.

```jsx
<div
    className={classNames("spectrum-Dialog-wrapper spectrum-Underlay", {
    "is-open": isModalOpen
    })}
    onKeyDown={handleKeyDown}
>
```

```javascript
const handleKeyDown = event => {
    // this function handles the events onKeyDown
};
```

## Keeping the Focus Within the Dialog Modal
If you noticed while tabbing inside of the dialog, it is possible to tab way out of the window itself and cycle through the browser even. We don't want that to happen since the dialog should contain its own tab sequence. We will need to trap the user's focus within and control the tab order. n order to do that, we would have to take away the native event behavior that goes on in the event handler `handleKeyDown`, by adding an `event.preventDefault()`, but only if the user is not hitting Tab or Escape:

```jsx
// event.keyCode
const Tab = 9;
const Escape = 27;

const handleKeyDown = event => {
    if (![Tab, Escape].includes(event.keyCode)) return;
    event.preventDefault();
};
```

This way, we can keep the default event handling for all the other keys like `Enter` or `Space`, but we're going to implement `Tab` and `Escape` manually, since we are modifying their behavior for this modal dialog.

## `Escape` Key
Let's implement the `Escape` key, which is supposed to close the dialog. We'll add to our `handleKeyDown`. It should check for an Escape keycode. When it is pressed, it calls `closeDialog()`:

```javascript
const handleKeyDown = event => {
    if (![Tab, Escape].includes(event.keyCode)) return;
    event.preventDefault();

    if (event.keyCode === Escape) {
        closeDialog();
        return;
    }
};
```

## `Tab` / `Shift` + `Tab` Keys
Next, let's add some conditionals for checking if the key(s) pressed are `Tab`, or `Shift` + `Tab`. The way pressing both `Shift` + `Tab` work for the event handler is that it will be registered as both `event.keyCode` (9) and `event.shiftKey`, so we would have to check for both `Shift` + `Tab` as a combination of `Tab` <strong>AND</strong> `Shift`. On the flip side, that means checking for `Tab` would be a `Tab` AND NOT `Shift` conditional, since `Shift` + `Tab` is still a `Tab` in the end. The long way of writing this would be:

```javascript
if (event.keyCode === Tab) {
    if (event.shiftKey) {
        // this is Shift + Tab
    } else {
        // this is only Tab
    }
}
```

The cleaner way, if you want to avoid nesting too many if/else:
```javascript
if (event.keyCode === Tab && event.shiftKey) {
    // placeholder for Shift + Tab logic
}

if (event.keyCode === Tab && !event.shiftKey) {
    // placeholder for Tab logic
}
```

Since the logic is never both true, it would be unnecessary to put them in an `else if`. We'll add the logic after this one tidbit.

## Keeping Tabs on All Focusable Elements ;)
In order to know if the user is at the first or the last focusable element, we would first need to keep tabs on all the elements that are focusable within the dialog. There are two ways that I can think of to go about this. It's possible to use `document.querySelectorAll()` to find all the focusable elements (inputs, buttons, etc.) so we would have an iterable list of elements. The second way is using callback refs by putting a `ref` on all the focusable elements within the dialog and writing the callback function to add the element to the array of refs. I'm going go with the second way in this example since we are in React-land and refs are the preferred way when trying to access a DOM element. In the scenario where you have more than a few elements to focus on, adding `ref` to all the elements can be a pain, so I would recommend using `element.querySelectorAll()` in that case.

First, I would need to keep a ref for the array of refs that I will use to add my focusable elements into it:
```javascript
const focusableELements = useRef([]);
```

Next, on my two buttons, I will simply add `ref={setFocusableElements}` with `setFocusableElements` being the callback function I will use to add to the array of all the focusable elements.

The two buttons in JSX:
```jsx
<button
    className="spectrum-Button spectrum-Button--secondary"
    onClick={closeDialog}
    autoFocus
    ref={setFocusableElements}
>
    Also Agreed
</button>
<button
    className="spectrum-Button spectrum-Button--cta"
    onClick={closeDialog}
    ref={setFocusableElements}
>
    Agreed
</button>
```

The `setFocusableElements` callback function:
```javascript
const setFocusableElements = element => {
    if (!element) return;
    if (focusableElements.current.find(focusableElement => focusableElement === element)) return;
    focusableElements.current = [...focusableElements.current, element];
};
```

Callback refs have the [caveat of being called twice](https://reactjs.org/docs/refs-and-the-dom.html#caveats-with-callback-refs) during an update, so I usually short-circuit the function if the element is `null` (the first run) and if the element already exists inside the array of refs.

## `Tab` at the Last Element, `Shift` + `Tab` on the First Element
Now that we have all the refs in `focusableElements`, we can move on to implementing the logic for `Tab` and `Shift` + `Tab`. We said that if the user presses `Tab` on the last element, we'd bring them to the first, and if they presses `Shift` + `Tab` while they are on the first element, we would send them to the last element.

Let's first define the first and last element:

```javascript
const elements = focusableElements.current;
const firstElement = elements[0];
const lastElement = elements[elements.length - 1];
```

[`document.activeElement`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentOrShadowRoot/activeElement) is an easiest way to get the currently focused element natively.

In the scenario of `Shift` + `Tab`, we would first check if the active element is the first element. If it is, we should focus on the last element. Otherwise, since that we took away all the default behavior logic, the default `Shift` + `Tab` on a non-first element should bring you back to the previous element. We can get the previous element relative to the currently focused element with `document.activeElement.previousSibling`. We just need to call `.focus()` accordingly.

`Shift` + `Tab` logic written in code:
```javascript
if (event.keyCode === Tab && event.shiftKey) {
    if (document.activeElement === firstElement) {
        lastElement.focus();
    } else {
        document.activeElement.previousSibling.focus();
    }
}
```

For `Tab`, we would check if the active element is the last element. If it is, then we will focus on the first element. Otherwise, the default behavior we need to re-implement would be focusing on the next element, which can be found with `document.activeElement.nextSibling`.

`Tab` logic written in logic:
```javascript
if (event.keyCode === Tab && !event.shiftKey) {
    if (document.activeElement === lastElement) {
        firstElement.focus();
    } else {
       document.activeElement.nextSibling.focus();
    }
}
```

## Conclusion
Now you can write your own fully accessible, W3C compliant dialog modal given all the information, tools, tips, and techinques I've given you here and in [Part 1](https://www.yanandcoffee.com/2020/05/26/accessible-modals/)! Hopefully this will equip you with the knowledge to write dialog modals with accessibility in mind.

Here's the Code Sandbox that I worked with for this example:
<iframe
    src="https://codesandbox.io/embed/accessible-modals-part-2-pljct?fontsize=14&hidenavigation=1&module=%2Fsrc%2Fcomponents%2FDialog.js&theme=dark"
    style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
    title="Accessible Modals Part 2"
    allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
    sandbox="allow-autoplay allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>
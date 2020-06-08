---
title: Accessible Modals, Part 1 - Its Issues And The Solutions
description: Modals are a pretty common pattern in UI/UX design, but they also come with a lot of accessibility concerns if done incorrectly. Here's how you can do it right.
date: 2020-05-25 23:25:00
updated: 2020-06-05 4:12:00
tags:
- a11y
- accessibility
- modals
- focus management
- javascript
- React
- tabindex
- refs
---

Modals are a pretty common pattern in UI/UX design and most people will understand how to interact with a modal due to its pattern affordance (dark underlay, typically pops over, etc), but they also come with a lot of accessibility concerns if done incorrectly. This post will be about how to create an accessibility compliant dialog modal in React. I will not be going into detail on how to create your own modal, but I will talk conceptually about what it is (so you can create your own), the types of accessibility concerns that come with designing your own modal, and how to address those issues.

The [full example](https://codesandbox.io/s/accessible-modals-after-part-1-bnvuu?from-embed) used for this post is written in React. If you are using pure HTML/Javascript, the same concepts/techniques can still apply, but you would need to convert the implementation from one to another. For example, in JSX, `tabIndex` is camelcased, but in HTML, `tabindex` should be all lowercase. Some concepts may be explained in HTML for simplification/clarity purposes.

I will be following the dialog modal guidelines from the [W3 Docs](https://www.w3.org/TR/wai-aria-practices-1.1/#dialog_modal).

## Before We Get Started
Here's the SparkNotes* version of the W3 requirements:
- Users cannot interact with content outside an active dialog window, meaning they contain their own tab sequence regardless of the other outside element (there are some cases where clicking outside of the dialog will close the dialog)
- Once the dialog is open, keyboard interactions should be as follows:
  - `Tab` moves forward to the next tabbable element
    - if you are at the end, it moves to the first
  - `Shift + Tab` moves to the previous tabbable element
    - if you are at the first, it moves to the last
  - `Escape` closes the dialog
- In almost all circumstances, opening a dialog should autofocus to the first focusable element contained in the dialog*
  - If the dialog has limited interactions, focus should be on the most frequently used element such as the `OK` or `Continue` button
- When a dialog closes, focus should return to the element that invoked the dialog unless the work flow design makes more logical sense to set the focus elsewhere*

<small>* means it has been summarized and you should refer to the [docs](https://www.w3.org/TR/wai-aria-practices-1.1/#dialog_modal) if you want more detailed guidelines.</small>

## What Is a Dialog Modal & How Does It Work
If you understand what a dialog modal is and how it "works," you understand how to implement it and the accessibility problems that may come from it. In simple terms without thinking about any code, a dialog modal is simply a box on top of another box.

![Simplfied drawing of the modal UI](./modal-concept.png)

Initially, the modal is not on the page, so what does it mean for the logic? We hide the modal when the page loads.

![Simplified drawing of the page with a button](./page-by-itself.png)

We only show the button for triggering the modal. When the user clicks the button, or triggers the button via the keyboard, the modal shows up. There may be an underlay element and some CSS that needs to be added to make this UI experience look and feel better, but the concept of a modal is simply 1) the HTML for the modal and 2) the logic to show and hide it. This is our modal pattern.

## ARIA Attributes
For screen reader users, this is important. This is the dialog div wrapping the content:

![div receiving role=dialog](./role=dialog.png)

It should receive 1) `role="dialog"`, 2) `aria-modal="true"` if it is a focus trap and <strong>AND</strong> there is visual styling that obscures the content outside of it, 3) aria labeling for screen readers to announce the purpose of the dialog, e.g. the title with either an `aria-label` or a combination of `aria-labelledby` and an `id` on the (visible) dialog title.

`aria-label`:
```html
<div role="dialog" aria-label="Coffee is Great">
```

`aria-labelledby` and matching `id` on the (visible) dialog title:
```html
<div role="dialog" aria-modal="true" aria-labelledby="dialog-label" aria-describedby="dialog-content">
```

[`aria-describedby`](https://www.w3.org/TR/wai-aria-1.1/#aria-describedby) can also be used. It is similar to `aria-labelledby`, as they both should be on the element that wraps the main content they are describing. E.g. in the above example, `aria-labelledby` points to the heading title matching the `id`. `aria-describedby` should point to the text that describes the primary purpose of which it is on.

```html
<div role="dialog" aria-modal="true" aria-labelledby="dialog-label" aria-describedby="dialog-content" class="spectrum-Dialog spectrum-Dialog--medium">
    <div class="spectrum-Dialog-header">
        <h2 id="dialog-label" class="spectrum-Dialog-title">Coffee is Great</h2>
    </div>
    <div class="spectrum-Dialog-content">
        <p id="dialog-content">Coffee is a Chihuahua mix and the best dog ever with a {stars div} rating.</p>
    </div>
    <div class="spectrum-Dialog-footer">
        <button class="spectrum-Button spectrum-Button--secondary">Also Agreed</button>
        <button class="spectrum-Button spectrum-Button--cta">Agreed</button>
    </div>
</div>
```

If all of the above is done correctly, it should be organized correctly like this for the screen reader users:

![accessibility tab on chrome](./aria.png)

## Keyboard Accessibility
In order to understand how to make an accessible modal, you have to test your UI. Start from the URL bar of your browser and do not use your mouse at all. Use only `Tab` and `Shift + Tab` keys to navigate through the UI. For my demo, I have 3 buttons that all trigger a modal. The testing experience looks like this:

<figure>
  <video controls="controls" width="100%" height="100%" maxWidth="960" maxHeight="500" name="Video Name" src="./demo.mov"></video>
</figure>

I tabbed forward and backward, interacted with the button trigger and tested the modal tab flow as well.

[This](https://codesandbox.io/s/accessible-modals-before-nczzo?file=/src/components/ButtonsWithModal.js) _looks_ like we would be done if all we cared about was how a user interface looks. But once the modal is open, we can see there are a few accessibility issues.

### Accessibility Problem #1: Auto Focus Element Inside Modal
As a user, once the modal is open, my focus appears to still be on the button that I invoked the dialog with and not on the first available element or the most frequently used element.

![Image of focus issue with active modal](./modal_with_focus_bug.png)

Solution:
When the user invokes the dialog, the first element, the "Also Agreed" button, should be auto focused since we don't want the user to accidentally trigger the call-to-action without realizing it. This is as easy as adding `autoFocus` attribute on the button inside of the Dialog. The only gotcha is that `autoFocus` would only work if you are mounting the `<Dialog />` component at the time of the click since `autoFocus` attribute is [polyfilled in React](https://github.com/facebook/react/issues/11851#issuecomment-351672131) to call `.focus()` on the element that has this attribute due to browser inconsistencies with the html attribute `autofocus`.

In my demo, within my Dialog component, I would put the `autoFocus` attribute on the button:

```jsx
<button
    className="spectrum-Button spectrum-Button--secondary"
    onClick={closeDialog}
    autoFocus
>
    Also Agreed
</button>
```

This fixes the focus that was left outside of the modal (because you can only have one focus at a time), and it autofocuses on the first element when the modal shows up, telling the user where their focus is.

### Accessibility Problem #2: Tab Sequence Is Not Contained Within
After fixing the above scenario, as a user, once the modal is open and tabbing past all interactable elements within the modal, my focus goes to an element outside of the active modal. I can also interact with the elements outside.

![Image of accessibility issue with active modal](./modal_with_focus_bug2.png)

Solution:
When the user invokes the dialog, the buttons on the page need to be removed from the tab order of the page so that we can rely on the native sequential tab order to guide the user to the interactable elements available inside the modal. To remove an element from the sequential keyboard navigation order, we need to give the button elements a `tabIndex` of -1. Since we still want these buttons to be interactable after we close the modal, we will want to conditionally set this tabIndex value so that they aren't focusable when the modal is open, but when the modal is closed, they should be.

In JSX:
```jsx
<button
    onClick={onButtonClick}
    className="spectrum-Button spectrum-Button--primary"
    tabIndex={isOpen ? -1 : 0}
    id="button-1"
>
    <span className="spectrum-Button-label">Tell me more!</span>
</button>
<button
    onClick={onButtonClick}
    className="spectrum-Button spectrum-Button--primary"
    tabIndex={isOpen ? -1 : 0}
    id="button-2"
>
    <span className="spectrum-Button-label">Who's Coffee?</span>
</button>
<button
    onClick={onButtonClick}
    className="spectrum-Button spectrum-Button--primary"
    tabIndex={isOpen ? -1 : 0}
    id="button-3"
>
    <span className="spectrum-Button-label">The drink?</span>
</button>
```

> Avoid using tabindex values greater than 0 as it'll end up like the `z-index` battle

### Accessibility Problem #3: Closing the Modal
As a user, when I close the modal, my focus ends up going to the end of the page, which, if I tab enough, goes back to the URL bar of the browser.

Solution:
When the user closes the modal, the focus needs to end up back on the initial element that triggered the modal. We can do this by figuring out which button was clicked, by storing a state in React called `selectedButtonId` and then keeping an array of refs for all the buttons so that we can get the ref of the button that was selected based on the selectedButtonId.

First, we want to keep a state of the clicked button so we use `useState` hook:
```jsx
const [selectedButtonId, setSelectedButtonId] = useState();
```

We modify the onButtonClick handler that was on our buttons to set the selectedButtonId:
```jsx
const onButtonClick = event => {
    setIsOpen(true);
    setSelectedButton(event.target.id);
};
```

Then, we need to keep an array of button refs by using `useRef` hook and setting the button refs to be set via a callback ref function:
```jsx
const buttonRefs = useRef([]);
const setSelectedRef = (element, id) => {
    if (!element) return;
    if (buttonRefs.current.find(ref => ref.id === id)) return;
    buttonRefs.current = [...buttonRefs.current, { element, id }];
};
```

```jsx
<button
    onClick={onButtonClick}
    className="spectrum-Button spectrum-Button--primary"
    tabIndex={isOpen ? -1 : 0}
    id="button-1"
    ref={element => setSelectedRef(element, "button-1")}
>
    <span className="spectrum-Button-label">Tell me more!</span>
</button>
<button
    onClick={onButtonClick}
    className="spectrum-Button spectrum-Button--primary"
    tabIndex={isOpen ? -1 : 0}
    id="button-2"
    ref={element => setSelectedRef(element, "button-2")}
>
    <span className="spectrum-Button-label">Who's Coffee?</span>
</button>
<button
    onClick={onButtonClick}
    className="spectrum-Button spectrum-Button--primary"
    tabIndex={isOpen ? -1 : 0}
    id="button-3"
    ref={element => setSelectedRef(element, "button-3")}
>
    <span className="spectrum-Button-label">The drink?</span>
</button>
```

Next, we need to trigger the focus on the previously selected button by adding a `focusSelectedButton` function:
```jsx
const closeDialog = () => {
    setIsOpen(false);
    focusSelectedButton();
};
const focusSelectedButton = () =>
    buttonRefs.current
        .find(ref => ref.id === selectedButtonId)
        ?.element?.focus();
```
<small>* I used [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) in `focusSelectedButton()` here so that the UI does not break if it can't find the selected button element.</small>

This should focus on the previously selected button when the dialog closes.

## Conclusion
I walked through the most common accessibility issues that one would face when creating their own modal and how to address these issues with techniques such as controlling the sequential tab order with `tabIndex` ([tabindex](https://developers.google.com/web/fundamentals/accessibility/focus/using-tabindex)) and [programmatically managing focus](https://reactjs.org/docs/accessibility.html#programmatically-managing-focus) with React refs.

That's it for part 1! Part 2 will be focused on how we can enable `Tab`, `Shift + Tab` + `Escape` keys within an active modal to behave like the guidelines mentioned in the [SparkNotes section](#before-we-get-started). This is the working demo for the current state:
<iframe
    src="https://codesandbox.io/embed/accessible-modals-after-part-1-bnvuu?fontsize=14&hidenavigation=1&module=%2Fsrc%2Fcomponents%2FButtonsWithModal.js&theme=dark"
    style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
    title="Accessible Modals: After, Part 1"
    allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
    sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts allow-autoplay"
></iframe>

Stay tuned!
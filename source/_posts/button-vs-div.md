---
title: Button vs Div for Accessibility
description: A TLDR; post on the pros/cons of using a <button> vs <div role="button">.
date: 2020-08-10 01:15:47
tags:
- React
- a11y
- accessibility
- javascript
- buttons
---

The close button is a pretty typical, recognizable icon. This icon's perceived affordance is 1) to close items, such as windows, or modals, or 2) to delete or remove items, such as removing a To-Do or deleting a table row.

In code, we can implement this quickly with an img tag, which would make it _look_ like a close button, but a picture showing an image is cool,  but not accessible. 

<figure>
    <img src="./icons8-close-window-48.png" alt="Close Window icon" />
    <figcaption>This <a target="_blank" href="https://icons8.com/icons/set/close-window">Close Window icon</a> is by <a target="_blank" href="https://icons8.com">Icons8</a>.</figcaption>
</figure>

To make this accessibility-compliant, you have to make it a button, because the main functionality is tp trigger an event/action, and semantic HTML is the natural solution for creating web pages for everyone. In this scenario, `<button>` or adding `role=" button" ` to a div that has a background image of the close button could make the button accessible. 

## Why would you use div over button?
If you have a specific framework or button components that already use a div, it may be easier to keep the div and simply add `role=" button" ` to the component without breaking its existing styles. A good example of this would be this scenario. Let's say that this is a React component that is part of an old UI codebase that is shared across multiple parts of a product, and multiple teams use this. Also, your company uses another UI library that may have default `<button>` styles that would make `<CloseButton />` really awkward if it was converted to a button tag instead of keeping its div tag. This is the JSX that everyone else uses whenever a close button is needed:

```jsx
import styles from './CloseButton.css';

const keyHandler = (event, onClick) => {
    if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        onClick();
    }
};

export default function CloseButton(props) {
    return (
        <div 
            role="button" 
            tabIndex="0" 
            className={styles.closeButton} 
            onKeyDown={event => keyHandler(event, props.onClick)}
            {...props}
        >
            Close
        </div>
    );
}
```

You can tell here that I had to implement the Space/Enter functionality with `onKeyDown` to trigger the same way as an `onClick` because div's do not natively implement keyboard events at all. I also needed to use `event.preventDefault()`, because Space by default does trigger scrolling on a page, and it's not supposed to be a close button that also scrolls down on the page when you trigger it. Lastly, this button has `tabIndex="0"`, which makes this button focusable via keyboard.

That's a lot of attributes to make an accessible div button! But sometimes, this is a necessary evil. If you are writing a new component, you should use a `<button>` tag whenever possible. The HTML button will have native keyboard event support that triggers on Space and Enter (or return) key, and you won't have fiddle with `tabIndex` or `onKeyDown`. It comes right out of the box if you use semantic HTML :).
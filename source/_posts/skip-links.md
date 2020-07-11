---
title: Creating a <SkipLinks /> Component
description: An easy, reusable solution for adding skip navigation links to all your React apps
date: 2020-07-05 14:04:17
tags:
- React
- skip links
- hooks
- accessibility
---

[Skip Navigation Links](https://www.w3.org/TR/WCAG20-TECHS/G1.html) are links on a web page that help keyboard and screen reader users navigate to the main content when the main content isn't easily accessible. They can be visible or invisible, at the top of the page, at the bottom, or elsewhere when the UI is more complicated. I'm not going to talk about the use cases of when you should be using them since [WCAG 2.0](https://www.w3.org/TR/WCAG20-TECHS/G1.html) and [WebAIM](https://webaim.org/techniques/skipnav/) covers those topics pretty well. I will talk about my solution and how it can help you create skip links across your React apps in an easy, consistent, and reusable way.

The working demo/intro page for my `<SkipLinks />` component is available if you'd prefer to get right into the component and its usage:

http://www.yanandcoffee.com/react-skiplinks/

## The Component
There are a few ways to create and define the skip links, but it will be easier once you understand how the structure of the skip links should be like. The component itself isn't a complicated UI. It is pretty much a navigation component that more or less would look like this:

```jsx
export default function SkipLinks() {
  // if there isn't any skip links, don't render anything
  if (!skipLinks.length) return null;

  return (
    <nav>
      <ul>
        {skipLinks.map(link => (
          <li key={link.title}>
            <a href=`#${link.id}`>Go to {link.title}</a>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

Notice that I did not define `skipLinks` here because there are a few ways to do this. It can be defined as a `skipLinks` prop that gets passed in by its parent, or maybe `skipLinks` would be defined in a static constants file to be imported into the component. For now, that doesn't matter. All you need to know is that you need a list of skip links with the id of the element you want your users to skip to and the title of which the skip link should show to the user. That would be a list`[]` of objects`{}` that have the `id: '#element-id'` and `title: 'Element Title'` props.

## The CSS
Next, you will want to set up the CSS so that it will hide the skip links from the visible eye until it receives focus. [`display: none`](https://alistapart.com/article/now-you-see-me/) makes the content inaccessible by screen readers, so avoid that!

> Note: It is preferred that the links are visible, however, if you have a design that does not account for visible skip links, you can hide them from the visible eye. The [WCAG 2.0 Success Criterion 2.4.1](https://www.w3.org/TR/2008/REC-WCAG20-20081211/#navigation-mechanisms-skip) only require that they be visible when they receive focus. 

```css
.skipLinks a {
  position: absolute;
  left: -10000px;
  width: 1px;
  height: 1px;
  overflow: hidden;
}

.skipLinks a:focus,
.skipLinks a:active {
  left: 0;
  width: auto;
  height: auto;
  overflow: visible;
}
```

## Defining The Skip Link Content
Now that you know the bare bones of what skip links should look/feel like, I can talk about the implementation of handling how to define the skip links. It's possible to pass in a static list of links, but this feels like it could quickly become a maintenance nightmare. 

Instead, why not use a Markup-based solution that doesn't require too many implementation details since the nav can be generated (mostly) based on content that's already there? I can query the DOM for elements with the data-* attribute and define the titles in the HTML, similarly to how [Dummy Text](http://dummytext.com/) works.

I created a `useSkipLinks` hook that internally uses `document.querySelectorAll` to query for elements that contain the `data-skip-link` attribute, but not `<pre>` and `<code>` elements. After querying to get the NodeList, it will generate the skip links data structure that works with the `<SkipLinks />` component.

### `useSkipLinks` Hook:
```jsx
import { useState, useEffect } from "react";

export default function useSkipLinks() {
  const [skipLinks, setSkipLinks] = useState([]);

  useEffect(() => {
    const skipLinkElements = document.querySelectorAll(
      "[data-skip-link]:not(pre):not(code)"
    );
    const links = Array.from(skipLinkElements, skipLink => ({
      title: skipLink.dataset.skipLink,
      id: skipLink.id
    }));

    setSkipLinks(links);
  }, []);

  return { skipLinks };
}
```

Everything is contained within the `<SkipLinks />` component, and you wouldn't need to generate the list of elements in a specific data structure.

## Conclusion
You may have to make minor modifications to make my code work for your use case, but understanding what skip links are and how they can be defined will help broaden your understanding of how to make your own reusable component, skip links, or not. This component is now open-sourced on [Github](https://github.com/yanandcoffee/react-skiplinks) and written with Typescript for more robustness.
---
title: Accessible Navigation with Smooth Scrolling + Focus Management
description: Ever wonder how your users can navigate to a section smoothly and not lose their keyboard focus?
date: 2020-05-08 19:31:12
tags:
- a11y
- accessibility
- scrolling
- css
- javascript
- focus management
- React
---
## Scenario
This is a pretty simple and relatively common use scenario in accessibility. I am a keyboard user. I land on a website and I want to skip to the section that I'm interested in given a list of navigation links. Like this:

<figure>
  <video controls="controls" width="100%" height="100%" maxWidth="960" maxHeight="500" name="Video Name" src="./demo.mov"></video>
</figure>

[Skip links](https://webaim.org/techniques/skipnav/) are a similar concept so the same solutions may apply.

I mocked up these examples to demonstrate the possible solutions in pure JS, pure CSS/HTML, and React. Please note that I added a `tabindex=0` to the headings so that they are focusable in all of the examples to show where the focus would be after clicking on a nav link, but this is just for the demo and may/may not be necessary for your UI. In the scenario where you have more focsable content within each section, you may want to keep your user's focus there, but visibly not show the outline on the heading element. This can be done by adding `outline: 0` to the `<h2>` element.

## The Best Solution
[scroll-behavior: smooth](https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-behavior), which this blog uses, is the best solution, hands-down. It uses the native `href` on the anchor tag, which is native HTML that will bring you to the element that has the same matching `id`. However, it will not work with a sticky top nav because it cannot calculate the height of the sticky top nav. My example here will use a sticky side nav instead to demostrate its natural awesomeness:

<p class="codepen" data-height="500" data-theme-id="dark" data-default-tab="result" data-user="yanandcoffee" data-slug-hash="zYvPMyW" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="scroll-behavior: smooth, a pure CSS solution">
  <span>See the Pen <a href="https://codepen.io/yanandcoffee/pen/zYvPMyW">
  scroll-behavior: smooth, a pure CSS solution</a> by Yan Li (<a href="https://codepen.io/yanandcoffee">@yanandcoffee</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

It only requires one line of CSS:
```css
html {
  scroll-behavior: smooth;
}
```
and your navigational anchor tags will need to have the `href` pointing to a hash + target element id like so:
```html
   <ul class="nav">
      <li class="navItem">
         <a href="#section-1-heading" class="navButton">Section 1</a>
      </li>
      <li class="navItem">
         <a href="#section-2-heading" class="navButton">Section 2</a>
      </li>
      <li class="navItem">
         <a href="#section-3-heading" class="navButton">Section 3</a>
      </li>
      <li class="navItem">
         <a href="#section-4-heading" class="navButton">Section 4</a>
      </li>
      <li class="navItem">
         <a href="#section-5-heading" class="navButton">Section 5</a>
      </li>
      <li class="navItem">
         <a href="#section-6-heading" class="navButton">Section 6</a>
      </li>
   </ul>
```

If you are using React and React Router, you can try [React Router Hash Link](https://github.com/rafrex/react-router-hash-link) to achieve the same experience. If you are developing for a SPA, it may conflict with your router and you may/may not be able to use this solution.

## The Alternative Solution
The alternative solution if you cannot use the native solution, is a pure Javascript one. It requires rethinking the experience. Why not focus first, then scroll?  It turns out that focusing before or after the user's selection doesn't matter, it's 99.9% the same experience.

There is a [`preventScroll`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOrForeignElement/focus#Parameters) option in the [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOrForeignElement/focus) method that allows you to do that, which makes the solution look like this:

<iframe
  src="https://codesandbox.io/embed/focus-preventscroll-true-solution-pure-js-bm6fe?autoresize=1?fontsize=14&hidenavigation=1&module=%2Fsrc%2Fcomponents%2Fnavs%2FPreventScrollNav.js&theme=dark"
  style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
  title="focus(), preventScroll solution"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

The key is to trigger a `element.focus()` and a `window.scrollTo()` on the click of each navigational element.

The React version would require using refs: https://codesandbox.io/s/focus-preventscroll-solution-n3sie?from-embed

## Last Words
All of the example solutions provided here are lightweight and provide a nice user experience while keeping accessibility in mind. It doesn't require a whole library, or a ton of code!

For smooth scrolling, while it is a cool user experience for the modern web, please note that it is not always preferred and can cause more accessibility issues if the user has difficulty dealing with motion. Please consider using the `prefers-reduced-motion` in addition to smooth scrolling. This [post](https://joshwcomeau.com/react/prefers-reduced-motion/) explains accessible animations and the use of `prefers-reduced-motion` pretty well.

I'm still learning about accessibility, but I now have the habit of checking and building it into all of my work and you should too. The world deserves to be able to access the web and the internet pages that we build, and it relies on us, the engineers, to achieve that.
---
title: Accessibility is not a project to be completed
description: It's more than just tickets in the backlog. You're not solving for accessibility if you treat it like a project.
date: 2020-06-21 17:00:47
tags:
- accessibility
- a11y
- community
- non-technical
---

{% picture steps.jpg %}
{% endpicture %}

Accessibility is so much bigger than just a project to be finished and completed. Most websites and products weren’t originally built with accessibility in mind. And thanks to the growing amount of website accessibility lawsuits, it has now become a concern for most companies. [Agile epics](https://www.atlassian.com/agile/project-management/epics#:~:text=An%20agile%20epic%20is%20a,and%20to%20create%20a%20hierarchy.) are created, and teams are formed around addressing accessibility. But even as it is a growing concern for many companies, engineers lack the knowledge to write accessible UIs. I see engineers optionally choosing to pick up accessibility work and leaving the job to either the one person who seems to like accessibility or leaving it to another team and having it to be their role in addressing accessibility. I don’t think that’s the right approach. It’s not just about equipping a team or having one person pick up the knowledge and putting that team or person in charge of accessibility. “It’ll go away by the end of the year.” No. There is no “end” to accessibility work. It should be part of the World Wide Web. We all have to pick up and accept that accessibility is going to be part of our everyday workflow. It’s not optional, and we should all hold ourselves accountable to that.

## Two Steps Forward and One Step Back 
First off, accessibility does not directly contribute to the revenue of a business. It’s one of those things that “have to be done” because it hurts the business (if the product is inaccessible) in either the form of a lawsuit or they lose a subset of their users. Both of which are bad for businesses, and companies should find ways to enforce or teach accessibility to the larger group. Most of us have a considerable backlog of accessibility issues that need to be addressed. Yes, maybe forming a team around addressing accessibility is the right thing to do here. But there’s more to it than that.

> Accessibility cannot be an *afterthought* when we implement new features.

As an individual contributor, I want to implore you (designers, engineers, product people, etc.) to lead by example here. We don’t someday magically have a fully accessible product by completing all of the accessibility tickets in the backlog. The problem remains that we’re not all working toward including accessibility in our everyday workflows. Accessibility cannot be an *afterthought* when we implement new features. You cannot just fix one ticket, and then your next new feature breaks the entire flow. There is no end to that. Engineers like to treat tasks as problems to be fixed, but the “fix” here is behavioral. It’s what you do. You need to change the way you work to address accessibility in the present and in the future. Accessibility isn’t going away.

## Dear Designers / Product
I’ve seen design mocks with accessibility requirements written in the specs and ticket/task description. I particularly liked mocks/specs that included the WAI-ARIA role to reference. I remember one of my first accessibility tickets on my new team asked to include accessibility as a layout grid with the grid role of our preexisting card catalog pattern. That brought me to the [layout grids](https://www.w3.org/TR/wai-aria-practices/examples/grid/LayoutGrids.html) documentation and implementing the new feature with `grid`, `row`, and `gridcell` roles. It probably feels out of scope for a designer’s work. Maybe you would work with Product on this piece. Without someone defining the requirements early on in the feature, engineers are not going to implement accessibility if they have no knowledge or experience with it. The problem is not that they don’t want to, but not everyone knows where to begin.

Another way to guide the engineer along is to give them a keyboard navigational flow mock, which shows what the user’s tab order for the experience should be. Sometimes, when you are adding a new feature to a preexisting page, it’s easy to not think through the whole experience because you understand the product and experience so well. It’s important here to think about how the user would navigate to the search bar from the beginning to the end. Here’s an example of all the accessibility concerns that you would think through if you’re implementing a search bar to a preexisting page.

1. Can the user navigate to the search bar with keyboard navigation?
2. After entering the search query, 
   - Can the user navigate to the next item (assuming the first matching search result)?
   - Can the user delete the query and return to their previous state?
   - Can the user navigate away from the search bar before or after entering a search query?

Sometimes, we *assume* the experience when we define the feature because we have a mental image of how the product should work. Without thinking through and testing all of these use cases, we may miss some accessibility concerns.

## Dear Engineers
We receive mocks from designers, and we implement them. If the tab order is in the design mocks, we can implement that as well. But often, this is not the case, so we have to hold ourselves accountable for making sure that our feature is accessible with the entire flow. UI libraries usually provide all the correct aria attributes in their components, so you wouldn’t have to worry about writing the right aria roles and attributes. If you are writing your own component, you should bake the aria attributes into the component so that other engineers won’t have to implement them manually each time.

A good example would be the X icon button that visually means “Close” to the user, but on screen readers, this may be confusing. An `aria-label` labeling it as “Close” would help here. These are little things that an engineer could learn or look up. A robust UI library could help with these aria attributes for the most part.

Outside of using a UI library, you should also think about keyboard accessibility, which is about the entire flow of all the combined UI components. Keyboard accessibility is sometimes tricky, depending on how all the preexisting components play with one another. Sometimes, even the most robust components, when combined doesn’t make an accessible UI. We still need to do a quick 3–5 minute accessibility walk-through to make sure we didn’t break anything that worked before.

## Accessibility walk-through to Test Keyboard Navigation
This can be done by either design, engineering, or product. Start at the top of the feature. It may help to click first near the top of your UI if you don't have skip links implemented to skip to the part where you've implemented your feature. Using only `Tab`, `Shift + Tab`, `Enter`/`Space`, and `Esc` navigate through the interface. If it helps, you can think of it as a game. `Tab` moves forward and `Shift` + `Tab` moves backward. `Enter`, or `Space` should trigger the interaction of the element in focus. `Esc` should be a way to exit whatever interaction you just triggered (e.g., modals or tooltips). Interact with all the elements surrounding your new feature, including getting *to* and *back* from the feature. You can imagine this is how keyboard users navigate through the UI.

I'll give an example of how this would look like. Let’s say the new feature you’ve added is an icon button that triggers a tooltip on hover/focus. The key here is that you want to make sure your new shiny addition didn’t break preexisting accessibility. You can start at the top of the page and Tab to the icon. Make sure that the flow to get to the icon still works. When you are focused on the icon, it should show the tooltip. Tab past the icon and the tooltip should disappear. Shift + Tab to go back to it, and the tooltip should still appear. If this all works, then you’re done.

## How Can We Maintain All This?
Ideally, we'd want some sort of audit or integration tests in place to keep our accessibility fixes in place. [Lighthouse](https://github.com/GoogleChrome/lighthouse) is an excellent open-source tool created by the Google Chrome team to audit web pages for performance, SEO, accessibility, etc. If you set up automation to run Lighthouse as a node module in a shell script whenever a build runs, you could audit for accessibility coverage and get reports on how accessible your web pages are per build. This would be a great way to keep your accessibility coverage in check.

For testing keyboard navigational workflows, unfortunately, Cypress does not natively support the tab key. There is an [issue](https://github.com/cypress-io/cypress/issues/299) created on Github back in 2016. There are workarounds in that thread. As of June 2020, if your integration tests are written in Cypress, it seems like the approved workaround from the Cypress team would be the [cypress-plugin-tab](https://github.com/Bkucera/cypress-plugin-tab). If you set up your Cypress tests like how you would test in an accessibility walk-through with the keyboard, then you should be covered for any new additions to that page.

## Conclusion
There are two primary issues with the way we view accessibility. One is that people _do_ think accessibility is a good thing, but they don’t really do anything as a whole (designers, engineers, product, company, etc.) to accept accessibility as part of our WWW. Secondly, we treat accessibility like a laundry list of tech debt cataloged in the backlog instead of it having it as part of our everyday work life.

Accessibility is a problem to be solved by everyone working together every day to address this. Not just one ticket, or a backlog of tickets, but every person involved with developing a new feature or product on the team; designers create mocks for engineers, engineers working on a new feature or product, and product folks who define requirements and negotiate deadlines with the customers. We all need to accept that accessibility is a part of our job. It's not a one-and-done, and there's no finish line here. Everyone is running the marathon together.
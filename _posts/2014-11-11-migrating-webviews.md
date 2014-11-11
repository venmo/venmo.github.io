---
layout: post
title: "Migrating Webviews"
author: Julian Connor
---

For the last month, the web team has been diligently working on porting our webview code from one repository to another. Through this long and tedious process we've learned a lot about the code we used to write and the code we now write. The striking contrast reminds us that sustained piecemeal progress gives large code bases the opportunity to bloom.

In this blog post, I will go through two valuable lessons we learned:
 - You can't put toothpaste back in the tube.
 - Do one thing at a time.

#You can't put toothpaste back in the tube
Imagine you are a developer who is blazing his or her own trail through the unknown; in this case, an unfamiliar repository. Wouldn't you repurpose patterns and conventions in order to complete your task?

Of course you would. All engineers are guilty of copying code every now and then. Despite being efficient, code copying has the potential to propagate anti-patterns throughout your codebase.

Here's an example of how a Backbone Model anti-pattern got out of the tube:

```js
INSERT CODE EXAMPLE HERE
```

Once anti-patterns are out of the tube they become increasingly difficult to subdue as their footprint greatens over time.

#How to avoid needing to put toothpaste back in its tube
Define and document a set of interactions for the code base:
 - How do we run tests? Create builds? Install dependencies? Start the dev server?
 - Is there a style guide? How about a linting process?
 - etc...

Additionally, think long and hard about the foundational aspects of your repository:
Are we happy with the overhead of adding new files to the repo? (E.g., boilerplate, dependency management, pathing, etc..)
Is the code architecture sustainable? I.e., Do we foresee these patterns scaling well?

Foundations dictate the resilience of their burdens.

#Only refactor one thing at a time
It's easy to become distracted when interacting with large amounts of legacy code. In the majority of cases, flaws and anti-patterns will emerge, potentially leading to lengthy spirals of refactoring. 

Before getting started, it's important to clearly define the goal of the refactoring and stay within its bounds. Two rounds of well defined refactorings can be completed in a much more steadfast manner than one round of wavering changes and improvements.

#Conclusion
Code quality is a function of its development environment. Defining a strong safety net, style guides, and best practices will lead to self-curation and the propagation of code that the team loves.

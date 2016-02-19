# *WORK IN PROGRESS*
# *Easily write concise, DRY, maintainable and scalable SCSS code in a large repo*

---

Table of contents:
  * [Motivation](#motivation)
  * [The main problems with CSS at scale](#the-main-problems-with-css-at-scale)
  * [A holistic solution](#a-holistic-solution)
   * [Best practices to write DRY and scallable SCSS code](#best-practices-to-write-dry-and-scallable-scss-code)
   * [The root-selectors.scss library](#the-root-selectors-scss-library)
   * [A well configured SCSS linter](#a-well-configured-scss-linter)
   * [A grunt task to enforce namespacing](#a-grunt-task-to-enforce-namespacing)
  * [A comparison table to other css solutions](#comparison-with-other-css-solutions)
  * [Conclusion](#conclusion)

---

## Motivation
CSS (and SCSS) tends to get messy.<br>
In a small project with just a few contributors, one can probably just read through the whole css code, hack in desired changes, and chances are not much will break. It doesn't matter if the css is ugly - as long as it just works. A glimpse at the output in the browser will tell you if things are working.
<br>
A large scale front-end project, with dozens of contributors, thousands of files, components, abstractions and a huge number of possible UI states - requires a different strategy. In this case, the CSS code should be readable and easy for refactoring. And in many cases the person who is about to make changes is not the person who originally wrote the code.
<br>

---

## The main problems with CSS at scale 
  * CSS is all at the global namespace (just like javascript)
  * Refactoring is complex; Hard to eliminate dead code (unreachable selectors)
  * Hard to know the visual output just by looking at the code.
  * Bloated stylesheets may lead to performance problems.
  * Its hard to write scoped CSS.
   
--- 

## A Holistic Solution

---
#### The *most important* guideline
 * Depending on your choice of framework (Angular, React, Ember or any other), your project has a way it declares DOM. Could be a js component file, an SVG file, an [rt](https://github.com/wix/react-templates) file, haml file, plain html file, etc.
 * Stick to your project's components abstraction, and split all css to scss modules accordingly. For each file which creates DOM - create a corresponding scss file, with the same name.
 * e.g. For each file `filename.html`, create a `filename.scss` file next to it in the same folder just next to it.
 * `filename.scss` should contain all --->
 
  

   * Stick with your project's components abstraction, and split all css to scss modules accordingly.<br> e.g. ng-templates in an angular.js project, html/haml if you use those, [jsx](https://facebook.github.io/jsx/) or [react templates](https://github.com/wix/react-templates) files in a react project, etc. I will use rt files [react templates](https://github.com/wix/react-templates) in the examples below
   * For each file `filename.html` (or `filename.svg` or `filename.js` or `filename.haml` - however DOM is created), create a `filename.scss` file next to it in the same folder just next to it
   * `filename.scss` should contain all scss code which refers to any DOM defined in filename.html

---

#### Best practices to write DRY and scallable SCSS code
#### root-selectors-helpers.scss library to enhance expression ability
#### A well configured SCSS linter.
#### A grunt task to enforce namespacing.

---

## Comparison with other css solutions
  * BEM
  * OOCSS
  * SMACSS
  * SUITCSS
  * Atomic
  * Css Modules - extra precompiling needed, can't integrate into existing project
  
Main problems:
* Hard to integrate into an existing project which grew over time, expenssive refactors needed
* Templating system not be compatible with modules naming systems
* Extra syntax is introduced, need to learn and teach developers new stuff
* integrate 3rd-party css? forget it
* Extra semantics which intend to put things into order, are forigen to the project, and nobody cares about it when they dont last as the project grow


---

## Conclusion



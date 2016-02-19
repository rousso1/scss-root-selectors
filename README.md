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
####1. The *most important* guideline
 * Stick to your project's components abstraction, and split all css to scss modules accordingly. If a file creates DOM - create a corresponding scss file, with the same name. e.g. for file `filename.html`, create a `filename.scss` file next to it in the same folder just next to it.
 * Depending on your choice of framework (Angular, React, Ember or any other), your project has a way it declares DOM. Could be a js component file, an SVG file, an [rt](https://github.com/wix/react-templates) file, haml file, plain html file, etc.
 * `filename.scss` should contain all styling for `filename.html`, and only its styling.
 
---
#####2. Every scss file should have a *single* root selector matching the rt root element. 

Everything else should be nested. This selector should be unique at the application level, this is enforced with [a grunt task](#a-grunt-task-to-enforce-namespacing). e.g:

```scss
.my-comp {
   position: relative;
   height: 10px;

   > .label {
       font-size: 14px;
   } 
}
```

---
#####3. Styling nested components is done in the nested components corresponding scss file. 

e.g: If `my-comp.html` has a child component `child.html`, the styling of `child.html` is done in `child.scss` using a parent selector (this strictly follows [rule #1 above]()):
    
```scss
//child.scss:

.child {
    position: relative;

    //overrides:
    .my-comp & {
       //when .child has .my-comp as an ancestor, we want to style it differently:
       position: absolute; 
    } 
}
```
This probably looks like a bad seperation of concerns, and it probably bothers you that `.child` knows about `.my-comp` intimately. And thoughts about how does child.scss go open source from here, where every small component knows about all its overrides.
Well, currently all of these overrides exist in the project, only they are scattered around different files, probably organized by features, and not by DOM (as [rule #1]() says). So refactoring `.child` (e.g rename to `.boy`) is much harder if you have this css all around your project, where other components override it.
Moreover, consider a situation where `.my-comp` and `.my-comp2` both override `.child`. If those overrides won't be placed in the same place inside child.scss, most likely they will duplicate the styling and we'll result with more outputted css. In our case, we will just append `.my-comp2` to the existing selector:
```scss
.my-comp &, 
.my-comp2 & {
  position: absolute;
}

```
Going "open source" with child.scss


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



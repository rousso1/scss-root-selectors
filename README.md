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
 * Stick to your project's components abstraction, and split all css to scss modules accordingly.<br>If a file `filename.html` creates DOM - create a corresponding `filename.scss` scss file next to it, in the same folder.
 * Depending on your choice of framework (Angular, React, Ember or any other), your project has a way it declares DOM. Could be a js component file, an SVG file, an [rt](https://github.com/wix/react-templates) file, haml file, plain html file, etc.
 * `filename.scss` should contain all styling for `filename.html`, and only its styling (e.g not style any of its child components).
 
---
#####2. Every scss file should have a *single* root selector matching the root element of the html file. 

Everything else should be nested. This selector should be unique at the application level, this should be enforced with [a grunt task](#a-grunt-task-to-enforce-namespacing). e.g:

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

e.g: If `my-comp.html` has a child component `button.html`, the styling of `button.html` is done in `button.scss` (this strictly follows [rule #1 above](#1-the-most-important-guideline)):

If you have something smart to say semantically - define an API (e.g. additional defined classes `.warning`, `.error` for a button component),

If not, and you just want to change the button's style given it has a specific ancestor - use sass parent selector (&).
    
```scss
//button.scss:

.button {
    position: relative;
    
    &.warning {
      color: yellow;
    }
    
    &.error {
      color: red;
    }

    //overrides:
    .my-comp & {
       //when .button has .my-comp as an ancestor, we want to style it differently:
       position: absolute; 
    } 
}
```
This may look like a bad seperation of concerns, and it probably bothers you that `.button` knows about `.my-comp` intimately. And thoughts about how does button.scss go open source from here, and worries that button.scss will get bloated from all the possible overrides it has in the project.<br>
Well, all of these overrides anyways exist in the project, only they are scattered around different files, probably organized by features, and not by DOM (as [rule #1](#1-the-most-important-guideline) says).<br>
So refactoring the classname `.button` (e.g rename to `.something-else`) is much harder if you have this css all around your project, where many other components override it.
Moreover, consider a situation where `.my-comp` and `.my-comp2` both override `.child`. If those overrides won't be placed in the same place inside child.scss, most likely they will duplicate the styling and we'll result with more outputted css. In our case, we will just append `.my-comp2` to the existing selector:
```scss
//overrides:
.my-comp &, 
.my-comp2 & {
  position: absolute;
}

```
Going open source with button.scss just means we need to delete all the `//overrides` sections. Thats it.

---
#####4. Always prefer immediate child selector (>) over regular descendent selector: 
they perform slightly better, but more important: they prevent bleeding css to nested components!
    
```scss
//dont:
.parent-class {
    //dont:
    .child-class {
        color: red;
    }

    //do:
    > .child-class {
        color: red;
    }
}
```
Note: lots of nesting can lead to performance issues. As a rule of thumb - up to 5 nested levels is okay. If you have more - consider splitting your html file into smaller files.

---
#####5. Avoid tagName selectors, prefer a semantic class selector:

```scss
.component {
    //dont:
    span {
        font-size: 14px;
    }
    
    //do:
    > .button-label {
            font-size: 14px;
        }
    }
}
```

---
#####6. Avoid wildcard selectors (*) - they affect child components and bleed css.
```scss
//dont:
* {

}

//do: specify what you style
.selector1,
.selector2, 
.selector3 {
        //css here
}
```

---
#####7. Avoid negation pseudo selectors: `:not(.some-class)`
They are complex, hard to refactor, and there is (almost) always a more clear alternative

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



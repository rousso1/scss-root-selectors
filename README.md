
##Concise, DRY, maintainable and scalable SCSS solution for styling large scale projects — Edit


---
##The main problems with css at large scale apps:
   * CSS is all at the global namespace (just as js is)
   * Refactoring is complex; hard to eliminate dead code (dead selectors)
   * Hard to know the visual output just by looking at the code - usually need to open browser and inspect.
   * Hard to write scoped styling rules without affecting children, as it should be in a components-based application.
   * Bloated stylesheets may lead to performance problems.
   * BEM, Atomic, css modules - its not it.

---
##Ask yourself: are you happy with your project's css the way it is now?

---
The solution consists of 4 parts:
  * [Six guidelines for coding style](#guidelines)
  * [SCSS root-selectors lib](#expressing-a-component-conditional-statesmodes-with-regard-to-the-root-state-class---using-the-root-selectors-library)
  * [Grunt task](#grunt-task)
  * [scss-lint yml config file](#scss-lint-configuration)

---
##Guidelines
By following these guidelines, gain control over the css technical debt in your project. 
  * Scale as you grow
  * Refactor easily
  * Easy to step into someone else's code
  * Easy to integrate into an existing project

---
##The *most important* guideline
   * Split css to scss modules following the projects components abstraction (e.g. rt files)
   * For each file `filename.rt`, create a `filename.scss` file next to it in the same folder .
   * Only write style rules which refer to the DOM structure that belongs to the specific RT. 

---
##Best practices

---
#####1. One scss file per rt file (also per svg file, skin, or any other piece of dom/template file). 
e.g:
```bash
angle.rt
angle.scss
```

---
#####2. Every scss file should have a *single* root selector matching the rt root element. 

Everything else should be nested. This selector should be unique at the application level. e.g:

```scss
.control-angle {
   position: relative;
   height: 10px;

   > .control-label {
       font-size: 14px;
   } 
}
```

---
#####3. Styling nested components is done in the nested components corresponding scss file. 

e.g: `angle.rt` has a child component `stepper.rt`, and configuring its style is done in `stepper.scss` using a parent selector:
    
```scss
//stepper.scss:

.input-stepper {
    position: relative;

    //overrides:
    .control-angle & {
       position: absolute;
    } 
}
```

---
#####4. Always prefer immediate child selector (>) over regular descendent selector: 
they perform better, and they also help preventing bleeding css to nested components.
    
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
##Expressing a component conditional states/modes with regard to the root state (class) - using the root-selectors library 

---
* Given a simple component with some nesting:

```scss
.a-root-selector {
  > .a-first-child {
    > .a-second-child {
      color: blue;
    }
  }
}
```

---
If we want `.a-second-child` to receive `color: orange;` when the component has another root class. (e.g. `.show-on-all-pages`), we can use the ancestor selector (`&`):

```scss
.a-root-selector {
  > .a-first-child {
    > .a-second-child {
      color: blue;
      .show-on-all-pages & {
        color: orange
      }
    }
  }
}
```

---
It will give us the following css, where `.show-on-all-pages` is an ancestor or `.a-root-selector`:

```css
.a-root-selector > .a-first-child > .a-second-child {
  color: blue;
}

.show-on-all-pages .a-root-selector > .a-first-child > .a-second-child {
  color: orange;
}
```

---
If you need to have it on the same root selector, you'll need to use a mixin:
```scss
.a-root-selector {
  > .a-first-child {
    > .a-second-child {
      color: blue;
      @include when-root-has-class(show-on-all-pages) {
        color: orange;
      }
    }
  }
}
```

---
Will give you the following css (note there is no space)
```css
.a-root-selector > .a-first-child > .a-second-child {
  color: blue;
}

.show-on-all-pages.a-root-selector > .a-first-child > .a-second-child {
  color: orange;
}
```

---
Similarly, the following SCSS mixins are available for your use as part of the framework:<br><br>
Classes modes for a selector:<br>
* `when-root-has-class($class-name)`: when the specified classes is authored on the root level.
* `when-root-has-any-class($class-names)`: one of the specified classes (one or many) is authored on the root level
* `when-root-has-all-classes($class-names)`: all specified classes (one or many) are authored on the root level

---
Pseudo class modes for a selector:<br>
* `when-root-has-pseudo-class($pseudo-class)`: The specified pseudo class is authored on the root level
* `when-root-has-all-pseudo-classes($pseudo-classes)`: all specified pseudo classes are authored on the root level
* `when-root-has-any-pseudo-classes($pseudo-class-names)`: any of the specified pseudo classes are authored on the root level.

---
class and pseudo-class together<br>
* `when-root-has-all($class-names, $pseudo-class-names)`: all specified combination of class names and pseudo classes are authored on the root level.
* `when-root-has-any($class-names, $pseudo-class-names)`: any of the specified combination of class names/pseudo class names are authored on the root level.

---
Ancestor conditions:<br>
* `when-parent-has-pseudo($pseudo-classes)`: if previous part of selector has the specified pseudo classes, apply the styling.
* `when-ancestor-has-pseudo($pseudo-classes, $levels)`: when an ancestor up the selector has the specified pseudo classes, apply the styling. The ancestor is specified by an number of levels up the selector.
* see [root selector helpers](https://github.com/wix/santa-editor/blob/master/packages/baseUI/src/main/framework/rootSelectorHelpers.scss)

---
##Example: A custom checkbox 
  * Specified colors for various different modes (regular / hover / selected)
  * Needs to have a "beautiful" mode which specifies different styling.
(See the Pen <a href='http://codepen.io/eitaneitan/pen/zqeMZM/'>zqeMZM</a> by Eitan (<a href='http://codepen.io/eitaneitan'>@eitaneitan</a>) on <a href='http://codepen.io'>CodePen</a>.)

```html
<label class="control-checkbox">
  <input type="checkbox" class="input" />
  <div class="content"></div>
</label>
```

---
```scss
.control-checkbox {
  display: inline-block;
  cursor: pointer;

  > .input {
    display: none;

    + .content {
      width: 28px;
      height: 28px;
      background-color: red;
      border: solid 3px blue;
    }

    &:checked + .content {
      background-color: green;
    }
  }

  &:hover > .input + .content {
    background-color: yellow;
  }

  &.beautiful {
    > .input {
      + .content {
        background-color: cyan;
        border-color: lime;
        border-radius: 50%;
      }
      
      &:checked + .content {
        background-color: orange;
      }
    }
    
    &:hover {
      > .input + .content {
        background-color: grey;
      }
    }
  }
}
```

---
A better approach: DRY, concise, extensible, refactorable, DOM-like structure:
```scss
.control-checkbox {
  display: inline-block;
  cursor: pointer;

  > .input {
    display: none;

    + .content {
      width: 28px;
      height: 28px;
      background-color: red;
      border: solid 3px blue;
      
      @include when-parent-has-pseudo(checked) {
        background-color: green;        
      }
      
      @include when-root-has-pseudo-class(hover) {
        background-color: yellow;        
      }
      
      @include when-root-has-class(beautiful) {
        background-color: cyan;
        border-radius: 50%;
        border-color: lime;
        
        @include when-parent-has-pseudo(checked) {
          background-color: orange;
        }
        
        @include when-root-has-pseudo-class(hover) {
          background-color: grey;          
        }
      }
    }
  }
}
```

---
Compiled CSS Code:
```css
.control-checkbox {
  display: inline-block;
  cursor: pointer;
}
.control-checkbox > .input {
  display: none;
}
.control-checkbox > .input + .content {
  width: 28px;
  height: 28px;
  background-color: red;
  border: solid 3px blue;
}
.control-checkbox > .input:checked + .content {
  background-color: green;
}
.control-checkbox:hover > .input + .content {
  background-color: yellow;
}
.beautiful.control-checkbox > .input + .content {
  background-color: cyan;
  border-radius: 50%;
  border-color: lime;
}
.beautiful.control-checkbox > .input:checked + .content {
  background-color: orange;
}
.beautiful.control-checkbox:hover > .input + .content {
  background-color: grey;
}
```
---
##Grunt task 
ensure namespaces uniqueness in the project's SCSS files at build time
<br>TBD

---
##scss-lint configuration
These REALLY assist with getting things in order:
```ruby
  DeclarationOrder:
    enabled: true

  DisableLinterReason:
    enabled: true

  DuplicateProperty:
    enabled: true

  SingleLinePerProperty:
    enabled: true
    allow_single_line_rule_sets: false

  SingleLinePerSelector:
    enabled: true

  MergeableSelector:
    enabled: true
    force_nesting: true

  EmptyRule:
    enabled: true
```
---
##What did we achieve?
   * All css which affects a certain piece of DOM is always placed next to it in a dedicated scss file. Easy to find and change.
   * scss files become less complex, straight-forward and more performant.
   * Unifying similar style rules becomes easy; Its all in the same scss file.
   * All component style rules are nested in its selector. Nothing at the global scope.
   * When refactoring a react component (changing/deleting), its easy to see where all its css is at.
   * Easy to reuse an existing selector
   * Easy to code without opening a browser

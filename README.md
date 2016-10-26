##Express a conditional states/modes with regard to the root state (class)

###Motivation
---
Given a simple component CSS with some nesting:

```scss
.comp-root-selector {
  > .child {
    > .another-child {
      color: blue;
    }
  }
}
```

---
`.another-child` should get `color: orange;` when `.comp-root-selector` is appended with an additional class: `.special`. Let's examine with using the built in [ancestor selector](http://thesassway.com/intermediate/referencing-parent-selectors-using-ampersand) (`&`):

```scss
.comp-root-selector {
  > .child {
    > .another-child {
      color: blue;
      
      .special & {
        color: orange
      }
    }
  }
}
```

---
It will yield the following CSS, where `.special` is an ancestor or `.comp-root-selector`. Note the space between them:

```css
.comp-root-selector > .child > .another-child {
  color: blue;
}

.special .comp-root-selector > .child > .another-child {
  color: orange;
}
```

---
If you need to have it on the same root selector, use the library's mixin:
```scss
.comp-root-selector {
  > .child {
    > .another-child {
      color: blue;
      
      @include when-root-has-class(special) {
        color: orange;
      }
    }
  }
}
```

---
Will yield the following (desired) CSS. Note: there is no space.
```css
.comp-root-selector > .child > .another-child {
  color: blue;
}
.special.comp-root-selector > .child > .another-child {
  color: orange;
}

```

---
Similarly, the following SCSS mixins are available as part of the library:<br><br>
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
Attribute modes for a selector:<br>
* `when-root-has-attr($pseudo-class)`: The specified attribute selector is authored on the root level
* `when-root-has-all-attrs($pseudo-classes)`: all specified attribute selector are authored on the root level
* `when-root-has-any-attrs($pseudo-class-names)`: any of the specified attribute selector are authored on the root level.
* `create-attr-selector($name, $value)`: a shorthand for the [name="value"] attribute selector string. use as a constructor for convenience

---
Any combination:<br>
* `when-root-has-all($class-names, $pseudo-class-names, $attr-selectors)`: all specified combination of class names,  pseudo class names and attribute selectors, authored on the root level.
* `when-root-has-any($class-names, $pseudo-class-names, $attr-selectors)`: any of the specified combination of class names/pseudo class names/attribute selectors are authored on the root level.

---
Advanced: Ancestor pseudo conditions:<br>
* `when-parent-has-pseudo($pseudo-classes)`: if previous part of selector has the specified pseudo classes, apply the styling.
* `when-ancestor-has-pseudo($pseudo-classes, $levels)`: when an ancestor up the selector has the specified pseudo classes, apply the styling. The ancestor is specified by an number of levels up the selector.
* see [root selector helpers](https://github.com/wix/santa-editor/blob/master/packages/baseUI/src/main/framework/rootSelectorHelpers.scss)

---
##Example: A custom checkbox 
  * Specified colors for various different modes (regular / hover / selected)
  * Needs to have a "beautiful" mode which specifies different styling.
(See <a href='http://codepen.io/eitaneitan/pen/zqeMZM/'>Pen</a> by <a href='http://codepen.io/eitaneitan'>@eitaneitan</a> on <a href='http://codepen.io'>CodePen</a>.)

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
##Recommended [scss-lint](https://github.com/brigade/scss-lint) configuration
It really assists with getting things in order:
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


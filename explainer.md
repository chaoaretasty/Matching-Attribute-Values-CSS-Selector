# Matching dynamic attribute values in CSS

## Brief description
This is a proposal for a new CSS feature that targets elements with matching attribute values, where the values are not explicitly stated.

This feature can be used in many ways, but it'll be explained in the context of _selecting a child element, based on its parent's attribute value_.

_ex 1_
``` html
<div data-selected-value="1">
   <div data-value="1"></div> <!-- match! -->
   <div data-value="2"></div>
   <div data-value="3"></div>
</div>
```

## The problem

In _ex 1_, we can already target the matching element with this selector string:

_ex 2_
``` css
[data-selected-value="1"] [data-value="1"] {}
```

However, the syntax gets very clunky very quickly when we want other child elements to be selected by changing `data-selected-value` in js.

_ex 3_
``` css
[data-selected-value="1"] [data-value="1"],
[data-selected-value="2"] [data-value="2"],
[data-selected-value="3"] [data-value="3"] {}
```

The problem is that the CSS selector logic is static. If we want to add another child element with a `data-value` of `4`, it would require either adding another CSS rule dynamically, or doing it preemptively by adding an arbitrary amount of rules for more possible options in the future.

## Solution

Instead of needing to explicitly state every possible selection option like in _ex 3_, we can simply state that an element should be targeted if it has a matching attribute value, regardless of what that attribute value is.

### Proposed syntax

These are some syntax methods proposed over in [discourse.wicg.io/t/1687](https://discourse.wicg.io/t/selecting-matching-attribute-values-in-css/1687). The given examples have been changed to work with the structure of _ex 1_.

#### 1.
``` css
[data-selected-value=@value] [data-value=@value] {}
```
_[proposed by @tvler](https://discourse.wicg.io/t/selecting-matching-attribute-values-in-css/1687)_

#### 2.
``` css
:matches-ancestor-attribute(data-value, data-selected-value) {}
```
_[proposed by @rodneyrehm](https://discourse.wicg.io/t/selecting-matching-attribute-values-in-css/1687/11)_

#### 3.

``` css
[data-selected-value] {
  --selected-value: attr(data-selected-value);
}

[data-value=var(--selected-value)] {}
```
_[proposed by @rodneyrehm](https://discourse.wicg.io/t/selecting-matching-attribute-values-in-css/1687/11)_

### Review of the syntax

#### 1.
This example defines a placeholder variable (in this case `@value`) to act as the value for the attributes `data-selected-value` & `data-value`.

This is the shortest syntax proposal, but it introduces a new usage for the `@` symbol which could be confusing for developers. It also creates a new variable-like syntax in css which differs from css's already-existing variable syntax.

#### 2.
This exposes the feature as a pseudo-class, which feels like a good fit. However, it targets an arbitrary parent element & parent element selection in CSS doesn't exist yet.

#### 3.
This is my favorite solution. It's simple & fits in with already-existing css features (attribute selectors & css variables).

There are 2 current problems with this syntax though.

(1) The value of `attr(data-selected-value)` for `[data-selected-value]` isn't inherited by it's child elements – the actual string "attr(data-selected-value)" is passed down, which would produce a different value than expected if we were to infer that the scope of a variable value in a selector string would be its target elements.

(2) The css variable spec would need to be extended to allow variables to be called within a selector string.

## Final Solution
I currently think that syntax proposal 3 is the best fit for this feature. Here's a way to fix problem (1) I brought up in my review:

``` css
[data-selected-value] {
  --selected-value: attr(data-selected-value);
}

[data-selected-value]:var(--selected-value) [data-value=var(--selected-value)] {}
```

The newly proposed pseudo-class `var` would expose the computed value of its parameter within the selector string, making any use of `var(--selected-value)` equal `1`, in the case of my given example.

Extending css variables so they can be called outside of a CSS block can not only make this feature I'm proposing possible, it can make a huge assortment of complex CSS selectors like mine possible as well.

## Example

Here is a generic tab container where only one active tab can be visible at a time:

_ex 4_
``` html
<div data-active-view="1">
   <div data-view="1">1</div>
   <div data-view="2">2</div>
   <div data-view="3">3</div>
</div>
```

If we wanted to control the active view by changing the `data-active-view` attribute value, this is the CSS we would need today:

_ex 5_
``` css
[data-view] { display: none; }

[data-active-view="1"] [data-view="1"],
[data-active-view="2"] [data-view="2"],
[data-active-view="3"] [data-view="3"] {
   display: block;
}
```

This is the same behavior, but using the proposed CSS features:

_ex 6_
``` css
[data-view] { display: none; }

[data-active-view] {
  --active-view: attr(data-lol);
}

[data-active-view]:var(--active-view) [data-view=var(--active-view)] {
  display: block;
}
```

[Interactive codepen](http://codepen.io/anon/pen/NRajrR)
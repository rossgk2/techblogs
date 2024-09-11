# A rabbit hole

Recently, I was learning about some new Adobe software, and came across the line of code `import Theme from "@swc-react/theme"`. This quickly dropped me into the web development education rabbit hole...

- A quick search shows me that `"@swc-react/theme"` is [React Wrappers for Spectrum Web Components](https://opensource.adobe.com/spectrum-web-components/using-swc-react/).
- Another search shows that [Spectrum Web Components](https://opensource.adobe.com/spectrum-web-components/) is a particular implementation of Adobe Spectrum that uses [Open Web Components](https://open-wc.org/guides/developing-components/getting-started/)'s project generator. 
- What is [Open Web Components](https://open-wc.org/guides/developing-components/getting-started/)? Well, whatever it is, it relies on something called [Lit](https://lit.dev/docs/).
- What is [Lit](https://lit.dev/docs/)? It's a JavaScript library that relies on [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components).
- At the end of the rabbit hole, we learn that [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components) is a collection of modern HTML and JavaScript features that allow implementation of "components", which are modular, HTML-parameterizable pieces of a webpage that have their own associated HTML, JavaScript, and CSS. Components are typically implemented by more heavyweight frameworks such as React or Angular.

Of course, few of the clarifying details I've added in the above bullet points were clear to me during my initial time in the rabbit hole.

The following is an article that presents the relevant content from the rabbit-hole in a more foundational, "bottom-up" approach. 

# Web components

"[Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components) is a suite of different technologies [standard to HTML and JavaScript] allowing you to create reusable custom elements - with their functionality encapsulated away from the rest of your code - and utilize them in your web apps."

The "suite of different technologies" are the custom elements JavaScript API, the shadow DOM JavaScript API, and the `<template>` and `<slot>` HTML elements.

## Custom elements (JavaScript API)

The *custom elements* JavaScript API allows

* extension of built-in HTML elements, such as `<p>`, so that an extended HTML element can be used in HTML with code such as `<p is="word-counter">` . (The argument to `is` specifies which extension of `<p>` is used.) These are called *customized built-in elements*.
* creation of new HTML elements that have new tag names such as `<custom-element>`. These are called *autonomous (HTML) elements*.

A custom element is implemented as a class which extends either

* an interface corresponding to an HTML element, in the case of extending an existing HTML element

  or

* `HTMLElement`, in the case of creating a new HTML element

The class will need to implement several "lifecycle callback functions". The class, say `Cls`, is then passed to `window.customElementRegistry.define("my-custom-element", Cls)`.

## Shadow DOM (JavaScript API)

The *shadow DOM* JavaScript APIs allow "hidden" DOM trees, called *shadow trees*, to be attached to elements in the regular DOM tree. Shadow trees are hidden in the sense that they are not selected by tools such as `document.querySelectorAll()`. They allow for encapsulation because none of the code inside a shadow tree can affect the portion of the overall DOM tree that is its parent.

Shadow trees are effected by using

* `<template shadowrootmode="open"> </template>` in HTML

  or

* `const shadow = elem.attatchShadow({mode: "open"})` in JavaScript

## `<template>`

The `<template>` *HTML element* is not actually rendered by the browser. Instead, when `template` is the JavaScript `Element` representing a `<template>` HTML element (e.g. `const template = document.querySelectorAll("#some-template")`), we are expected to manually render\* `template.content`. This manual rendering is done by writing code such as `document.body.appendChild(template.content)`.

But- still- what good is this? At this stage, all we know about `<template>` is that use of it requires manually rendering HTML. It seems useless!

\*`template.content` is of type `DocumentFragment`, which is a data structure that represents `template.innerHTML`. You can read about a situation in which you would want to use `DocumentFragment` over `innerHTML` [here](https://coderwall.com/p/o9ws2g/why-you-should-always-append-dom-elements-using-documentfragments). It's not clear to me how using `DocumentFragment` is vastly superior to `innerHTML` in *this* scenario, but there is probably some small performance advantage.

### Slotting

`<template>` does become quite useful when it's paired with the `<slot>` element. The `<slot>` element allows us to define portions of the `<template>` inner HTML that are variable so that we can later "plug-in" custom HTML into those portions of the `<template>` inner HTML.

In order achieve this functionality of `<slot>`, we must actually use `<slot>` alongside custom element and shadow DOM concepts, as this was how   `<slot>` was designed to be used.

## Slotted custom elements

We now describe how `<slot>` is used with custom elements, the shadow DOM, and templates to implement a "slotted" custom element.

1. Include code such as

```html
<template id = "some-template">
	...
	<slot name = "some-slot"> default text </slot>
	...
</template>
```

in the HTML.

2. In the class that defines a custom element, write a constructor that creates a shadow tree by including  `const shadowRoot = this.attachShadow({mode: "open"})` in the constructor.

3. In same constructor, right after the creation of the shadow tree, set `template.content` to be the inner HTML of the shadow tree: `shadowRoot.attachChild(template.content.cloneNode(true))`.

(To see an example of this, inspect [this webpage](https://mdn.github.io/web-components-examples/element-details/) with your browser's development tools.)

We see that the three concepts of custom elements, the shadow DOM, and templates are all involved. (1) and (3) are about templates, (2) is about the shadow DOM, and (2) and (3) occur in the custom element's constructor!

But how does `<slot>` come into play? Well, suppose that a custom element called "some-element" is configured in the above way. Then then the HTML

```html
<some-element> </some-element>
```

is interpreted by the browser to be the inner HTML of the template *with the inner HTML of the template's `<slot>` element replacing the template's `<slot>` element*. So, the browser will render the HTML

```
...
default text
...
```

Alternatively, the HTML

```html
<some-element>
	<div slot = "some-slot"> replacement text </div>
</some-element>
```

is interpreted by the browser to be the inner HTML of the template *with the inner HTML of the newly specified `<slot>` element replacing the template's `<slot>` element*. So, the browser will render the HTML

```
...
replacement text
...
```

# Modern components

The type of custom element above implements the idea of a *modern component*, which is 

- easily reusable
- encapsulated (in the sense that one component's code is separate from other components and does not affect other components state or behavior)
- allows for parameterization of HTML with `<slot>`

We've seen that writing the above type of custom element requires a lot of boilerplate. We could eliminate the boilerplate by writing a class that implements the modern component functionality. The class's constructor would take the HTML that is to underlie the modern component as an argument\*.

\* If `<slot>` functionality is used, then the HTML that is to underlie the modern component would contain a the same kind of `<slot>` element that `<template>` did above.

## Lit

[Lit](https://lit.dev/docs/) is a library that provides a class, `LitElement`, that implements this notion of modern component. As Lit's documentation says, the advantage of this approach is that, since modern components rely on standard HTML and JavaScript APIs, they are supported by almost all web browsers (all web browser that support the required HTML and JavaScript APIs, that is), and do not require any frameworks such as Angular or React to run.

### Open Web Components

[Open Web Components](https://open-wc.org/guides/) is a website that "gives a set of recommendations and defaults" on how to write modern web components". The "Getting Started" page recommends that to begin developing a web component, you should make use of their npm package by running `npm init @open-wc`, which generates an example Lit component.

# Spectrum Web Components

[Spectrum Web Components](https://opensource.adobe.com/spectrum-web-components/) is the "frameworkless" or "as close to vanilla JS as possible" implementation of Adobe Spectrum. Spectrum Web Components are Lit components and thus extend `LitElement`.

## React Wrappers for Spectrum Web Components

"[swc-react](https://opensource.adobe.com/spectrum-web-components/using-swc-react/) is a collection of React wrapper components for the Spectrum Web Components (SWC) library, enabling you to use SWC in your React applications with ease. It relies on the @lit/react package to provide seamless integration between React and the SWC library."

### Why not just use React Spectrum components?

swc-react and React components are two technologies that implement the idea of a component in some way. I would think that if we're using React, wouldn't it more natural to just use React components, and not import an extra library that make Lit components useable in React? Well, [Adobe documentation](https://developer.adobe.com/express/add-ons/docs/guides/design/user_interface/#:~:text=We%20recommend%20using%20swc%2Dreact,and%20is%20more%20actively%20supported.) says:

> We recommend using [**swc-react**](https://opensource.adobe.com/spectrum-web-components/using-swc-react/) over [React Spectrum](https://developer.adobe.com/express/add-ons/docs/guides/design/user_interface/#react-spectrum) in your add-ons based on React, because it currently offers a more comprehensive set of components which provide built-in benefits as detailed above in the [Spectrum Web Components section](https://developer.adobe.com/express/add-ons/docs/guides/design/user_interface/#spectrum-web-components), and is more actively supported.

So I suppose that answers my question :)
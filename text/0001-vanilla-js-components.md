# Vanilla JavaScript components
- Start Date: 2019-11-21
- RFC PR: (Leave empty—will fill in once PR exists)
- Rivet Issue or PR: (Leave this empty—will fill in with implementation PR if accepted)

## Summary
This RFC proposes a fundamentally new strategy for the way Rivet JavaScript is written that will hopefully solve some of the recurring issues we’ve seen with the way the Rivet 1.x.x JavaScript was originally written. By _Vanilla JavaScript components_ here we mean JavaScript components that:

- are written in plain JavaScript and that don't depend on UI libraries like React, Vue, etc., but that can work alongside those libraries if needed.
- are written as ES6 modules and are distributed as a bundle usable with modern build tools (Webpack, Rollup, etc.), or in a `<script>` tag as a browser bundle.
- provide developer conveniences found in other UI component-based models e.g. callback functions and custom events that can be used as hooks to extend functionality

## Motivation
The original design of the Rivet JavaScript components/widgets was focused on ease of use. The goal was to make it so that if developers dropped the script into their page and added the correct markup then everything would _just work_. After the initial release it soon became obvious that developers needed/wanted to be able to use Rivet's JavaScript components along side other libraries and control them via their own scripts. Rivet's original set of JavaScript widgets failed pretty noticeably at this.

While they were mostly rewritten in the [1.1.0 release](https://github.com/indiana-university/rivet-source/releases/tag/v1.1.0) to help fix some of these issues, there are still many areas where they fall short when trying to use them alongside other scripts—especially in situations where the DOM is changing and updating without a full page refresh as is common in most single page applications. This proposal outlines a new strategy for writing Rivet JavaScript components that will bring a number of the following benefits.

- More resilient and predictable functionality in browser environments where the component doesn't have any kind of exclusive knowledge of the DOM. For example, where DOM nodes are added after initial page load, or where the DOM is built by another JS library.
- The ability to create multiple instances of any component e.g. `const myDropdownInstance = new Dropdown()`, allowing components to maintain their own state (if needed) and developers to have access to life-cycle methods for each instance, e.g. `myDropdownInstance.destroy()`, `myDropdownInstance.init()`
- The ability to pass event-based callback functions as options to the instance:
    ```js
    const myDropdownInstance = new Dropdown(element, {
      onOpen: function onOpenCallback() {
        // Do stuff when dropdown opens
      }
    })
    ```
- More modularity and flexibility so that developers can use components however they like. We can ship a bundle of ES6 modules that developers can import on a component-by-component basis if they are using module bundlers like Webpack, Rollup, etc. For example, `import Dropdown from 'rivet-components';`. We can also ship a browser bundle that developers can use in a normal `script` tag e.g. `<script src="./path/to/rivet-components.js"></script>`.

## Detailed design
Let's consider the `Dropdown` example we've looked at already. Here's how the current Rivet Dropdown implementation looks.

```js
var Dropdown = (function(){
  // Implementation
  
  // 'Public' methods
  return {
    init: init,
    destroy: destroy,
    open: open,
    // more methods
  }
})();

// Later on we initialize the Dropdown
Dropdown.init();

/**
 * Or maybe call one of it's methods in our script passing it the id
 * of one the dropdowns in the DOM.
 */
Dropdown.open('my-dropdown');
```

There are a number of issues with the current design:

1. One single instance of the dropdown that requires lots of tricky DOM manipulation and event delegation to defend against updates and changes to the structure of the DOM.
2. The inability to create and destroy separate and unique instances of the dropdown.
3. No event callback options/hooks like the `onOpen` example mentioned above.
4. Everything is one browser global. It's not possible to `import` individual components as ES modules.

### Constructors FTW!
Now consider how we might write our new Rivet Vanilla Javascript components. We'll use the Dropdown example again. I'm using [ES6 classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) here for syntactic sugar, but we could potentially solve the same issues using regular functions and [JavaScript's prototypal inheritance](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain).

```js
// rivet-dropdown.js
export default class Dropdown {
  constructor(element, options) {
    // Set up details
  }

  init() {
    // setup event handlers, etc.
  }

  destroy() {
    // clean up
  }

  someOtherMethod() {

  }

  // More implementation details
}
```

Then later use the ES module bundle in another script.

```js
// my-script.js
import Dropdown from 'rivet-components';

const dropdownElement = document.querySelector('[data-dropdown="my-dropdown"]');
const myDropdownInstance = new Rivet.Dropdown(dropdownElement, {
  onOpen: function() {
    // Do something on open
  }
});
```

### Namespace for browser bundles
We should aim to ship a regular browser [iife](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) that is bundled under a single namespace, e.g. `Rivet`. Doing so will make sure that we avoid any future conflicts with other browser globals we don't know about that might share variable names used for common UI components like `Dropdown`, `Modal`, etc.

```html
<!-- index.html -->
<script src="./path/to/rivet-browser.js"></script>

<script>
  const myDropdownInstance = new Rivet.Dropdown(
    document.querySelector('[data-dropdown="my-dropdown"]')
  );
</script>
```

### Using with other JS libraries
Writing Vanilla components this way moving forward will also make it easier to wrap them in other UI libraries/frameworks if needed. Example: using the life-cycle methods of a React component.

```js
import React from 'react';
import { Modal } from 'rivet-components';

class App extends React.Component {
  constructor() {
    super();
    this.modalRef = React.createRef();
  }
  
  componentDidMount() {
    const wrappedModal = new Modal(this.modalRef.current, {
      // other options
    });
  }
  
  componentWillUnmount() {
    wrappedModal.destroy();
  }
  
  render() {
    return(
      <div className="rvt-modal" ref={this.modalRef}>
        // Modal markup ...
      </div>
    );
  }
}
```

### DOM Interaction and Manipulation
One of the recurring issues we've seen with the Rivet 1.x.x JavaScript widgets is the way the DOM set up and manipulation is handled. One of the aims for the new Vanilla JavaScript component should be to address this issue by:

1. Giving developers more control over when event listeners are attached and what elements they get attached to.
1. Designing components so that they are more "aware" of changes that might be happening to the pieces of the DOM they interact with. E.g. What happens when the dom gets updated by an outside script or library? How do our Vanilla JavaScript components check for those changes? Can they or should it be the responsibility of the developer using the component to notify the component of changes and use the provided component's available methods to respond appropriately? It would definitely be worth looking into [this when-elements library](https://github.com/indiana-university/when-elements) to potentially provide a mechanism for watching and responding for changes.

### Progressive enhancement
The current version of the Rivet JavaScript widgets were built to function assuming that `rivet.js` is available and loaded in the browser. That means that on a slow connection, or in the event that JavaScript fails, the widget becomes unusable, and generally speaking the content inside those widgets (like modals and dropdowns) becomes inaccessible because we are generally using CSS to hide content on load by default.

For example, we use CSS to hide content with the `aria-hidden="true"` attribute. If the CSS loads before the JavaScript that content is inaccessible/unreachable until JavaScript becomes available. And in the event that the JavaScript fails to load the content will never be reachable.

Instead of setting the initial state of our components via the HTML attributes that are present on initial page load, we should explore adding those attributes via the components JavaScript that way if the script fails to load or is slow, the content is still visible/available. Once the script is loaded, it will then apply the appropriate state using `setAttribute` API.

#### Potential progressive enhancement stumbling blocks
There are a couple of potential pain points we could see implementing progressively-enhanced JavaScript components this way.

1. A flash of un-styled content that becomes hidden after the JavaScript is loaded will be visible on slower connections.
1. There will be an increased amount of engineering effort to write components this way.

### Manual vs. automatic initialization
With the current version of Rivet all JavaScript widgets are automatically initialized on page load. All developers have to do is add the correct markup to their page, and all event listeners/handlers will be set up automatically for every instance of the corresponding markup on the page that uses the appropriate data attributes. For example, every instance of markup with `data-dropdown="..."` will work automatically as a dropdown menu.

With the new Vanilla Components proposal each instance of a component would need to be created manually as mentioned in section above on constructors.

```js
const dropdownElement =
  document.querySelector('[data-dropdown="my-dropdown"]');

const myDropdownInstance = new Rivet.Dropdown(dropdownElement, {
  onOpen: function() {
    // Do something on open
  }
});
```

While this provides the most amount of flexibility across the various use cases laid out in the previous sections, there may be a lot of developers that prefer the "plug-and-play" functionality of all widgets being automatically initialized on page load.

#### Auto-initialization functionality
As a part of this change we will need to explore an _Auto-initialization_ functionality that would let developers opt into all components with the correct markup being auto-initialized on page load, similar to existing `rivet.js` functionality.

This functionality needs more research, but at first thought it may require some combination of the following.

1. Shipping the pre-built/browser iife bundle with the auto-init behavior by default.
1. Exposing an `autoInit` property on the namespaced iife bundle/object. For example setting `Rivet.autoInit = false` after the pre-built browser bundle is loaded would disable auto-initialization and require developers to manually create all instances of components.

Example:

```html
<!-- in your HTML file -->
<script src="./path/to/rivet-browser.js"></script>

<script>
  Rivet.autoInit = false;
  
  // Now it's up to me to initialize all of my own components.
  const signUpModal = new Rivet.Modal(
    document.querySelector('data-modal="sign-up-modal"')
  )
</script>
```

### Programmatic control
For every Vanilla JavaScript component we should aim to provide developers with programmatic control of the component's interactive behaviors via methods on the component instance. Let's look at our Dropdown component again.

Generally speaking, the markup for a dropdown menu would consist of the menu itself containing a number of interactive menu items (button, links, etc.) and some other element to trigger the opening and closing of the menu—typically another HTML `button` element. In this scenario event handlers would be attached to the toggle button that controlled the opening and closing of the menu.

When we're thinking about a new Vanilla component model, we should always expose methods on a component that developers can use to control its functionality without requiring a specific DOM structure. Developers should be able to initialize the **dropdown menu element itself** in the app or site and control it however they like.

```html
<button id="controlled-dropdown-toggle">Dropdown toggle</button>
<div role="menu" id="controlled-dropdown">
  <button>Menu item one</button>
  <button>Menu item two</button>
  <button>Menu item three</button>
</div>

<script>
  const dropdownEl =  document.getElementById('controlled-dropdown');
  const myDropdown = new Rivet.Dropdown(dropdownEl);
  
  const myDropdownToggle =
    document.getElementById('controlled-dropdown-toggle');
    
  myDropdownToggle.addEventListener('click', () => {
    /**
     * Check if the menu is open based on some internal state of 
     * the instance and then react accordingly. (partial pseudo code here)
     */
    if (myDropdownToggle.menuIsOpen) {
      myDropdown.open();
    } else {
      myDropdown.close();
    }
  });
</script>
```

#### Modal Example
Another good example of the importance of programmatic control would be modals. Oftentimes a single modal might need to be triggered by multiple events and/or conditions. For example, if a user's session has expired and you would like to show a confirmation modal. There would be no need to have any controls available in the UI that would open the modal, so it should be able to be fully controlled via another script.

```js
const logoutModalInstance = new Rivet.Modal(
  document.getElementById('logout-modal')
);

// Somewhere else in the script.
function logOutUser(user) {
  user.loggedIn = false;
  logoutModalInstance.show();
}

// Set a timeout for 10 minutes once the user is logged in.
setTimeout(function() {
  logOutUser();
}, 10000);
```

#### Default to automatic control
By default we should offer a convention that allows developers to automatically wire up connections between DOM elements using data attribute/id combinations. This would happen by default as described in the [Auto-initialization functionality section above](#auto-initialization-functionality).

```html
<!--
  Using a [data-dropdown-toggle] attribute ties the open functionality
  to a dropdown element with the corresponding [data-dropdown] attribute.
-->
<button data-dropdown-toggle="auto-dropdown">Dropdown toggle</button>
<div role="menu" data-dropdown="auto-dropdown">
  <button>Menu item one</button>
  <button>Menu item two</button>
  <button>Menu item three</button>
</div>

<script>
  // Assuming the markup above, this dropdown would automatically "just work".
  const myDropdown = new Rivet.Dropdown(
    document.querySelector('[data-dropdown="auto-dropdown"]')
  );
</script>
```

### Custom events
This proposal talks a bit about event-driven callbacks like `onOpen`, `onClose`, etc. It seems like there is some value in being able to listen for custom events in your own scripts and react accordingly. Especially the ability to use `preventDefault()` to catch events and cancel them based on some outside application logic.

```js
import { Modal } from 'rivet-components';

const myModalInstance = new Modal(
  document.querySelector('[data-modal="my-modal"]')
});

function modalOpenHandler(event) {
  if (!someLocalAppCondition) {
    myApp.respondToModalOpening();
  } else {
    // If not, catch the event and bail out.
    event.preventDefault();
  }
}

document.addEventListener('rvt:modalOpen', modalOpenHandler);
```

## How we teach this
This new Vanilla components model will be a pretty drastic departure from the way Rivet's JavaScript widgets currently work, so there will be a slight learning curve.

We should be able to smooth over the pain of transitioning to this new model by offering the auto-initialization mentioned in the previous section. The key will be thoroughly documenting how to use the Vanilla components in _auto_ or _manual_ mode.

### Axioms and shared language
- By _**auto**_ mode we mean the default browser bundle that automatically initializes all Vanilla JS components on load.
- By _**manual**_ mode we mean that developers will be responsible for creating the instances of the Vanilla components as and when they need them. For example when wrapping them in the framework of their choice using that framework's life-cycle methods to create and destroy instances.

## Drawbacks
The main drawback to this proposal would be that this new model would cause a breaking change to current APIs. Although, as we already are planning to move toward a 2.0 release with other breaking changes and major updates, breaking changes in the JavaScript APIs are less of a concern.

### Increased complexity
Another potential downside would be the increased complexity in how the JavaScript components can be used. While we believe it would be a welcome change for a lot of developers to have more flexibility in the way they can use Rivet's JavaScript, the increased complexity may be a turn-off to other developers. We should be able to help ease the burden of additional complexity by offering the _auto_ mode [outlined in the "Auto-initialization functionality section](#auto-initialization-functionality) of this proposal.

## Alternatives
The only other real alternative is to leave things the way they are and do our best to improve the existing Rivet JavaScript components. After two-plus years of use in a fair number of production apps and sites, the need to evolve the way we approach Rivet's JavaScript offering seems apparent. At some point we will have to introduce breaking changes. We may as well make these improvements when we do.

## Unresolved questions
There are a few things that need some discussion and refinement, e.g. digging into the _auto_ initialization functionality.

### Base component class
If we go the route of using ES `class`es for our Vanilla components, would it make sense to try and identify some sort of base class we could abstract common functionality into?

```js
// rivet-base-component.js
export default class RivetBaseComponent {
  constructor() {
    this.someSharedProperty = true;
    this.someOtherSharedProp = [
      'one', 'two', 'three'
    ];
  }
  
  someCommonMethod() {
    // implementation....
  }
}
```

```js
// rivet-modal.js
import RivetBaseComponent from './rivet-base-component';

class Modal extends RivetBaseComponent {
  constructor() {
    super();
  }
  
  someCommonMethod() {
    super.someCommonMethod(); // Call the the parent method
    // And then do other stuff
  }
  // More implementation
}
```
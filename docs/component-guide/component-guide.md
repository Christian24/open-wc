# Component guide
This guide is meant to collect practices for building reusable custom elements. 

## Naming custom elements
To define a custom element you need a tag name which identifies it to the browser. Custom element tag names need to contain a hyphen (`-`). You can use this to your advantage to identify elements that belong to your project. 

For general purpose elements a prefix for the organization can be used and then the suffix can be the element's name. Example general elements of organization could be called `org-button` or `org-dialog`.

For specific projects or a prefix identifying the project might be used. For example for an admin interface a component might be called `adm-avatar`. We still intend to reuse `adm-avatar`, but it is definitely not as general purpose as a button. This would also allow us to have different versions of the avatar element: For example `prt-avatar` might be a custom element that might be used in the customer facing UI.

## The API of your elements
The API of your custom element is very important part of it and you should put great effort into making it as understandable and easy as possible, because otherwise nobody will use your elements.

### Follow the footsteps of the HTML specification
Naming things is hard. It is especially hard to name attributes, properties, methods and events. Luckily, there is an API people should already be familar with: The API of builtin elements defined in the HTML specification. When designing your API try to follow the path defined by builtin elements. 


### Attributes and properties
For attributes and properties: Your `org-textfield` might accept a boolean attribute called `required`, just like `input` does. If developers want to mark an element as required in a form they can use `required` or `required=true` just like they could use with builtin elements. 

However, try to stick to the minimum necessary, you do not want to reimplement builtin elements with different styling. Build only what you need. There is a good chance your `org-textfield` is just as useful without the `size` attribute defined in the HTML specification. 

Be careful not to overwrite properties and attributes defined in `HTMLElement`. This might lead to undesired consequences: For example an attribute or property `title`, might sound like a great name for a heading while the HTML specification has intended a different use for it (https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/title).

### Methods

The same is true for methods. You should try to follow the HTML specification where it makes sense. `org-textfield` might need a `checkValidity()` (for more information check https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5/Constraint_validation) just as builtin form elements. If the method is not defined in `HTMLElement` it might sometimes make sense to deviate from the original signature: You could change parameters or the return type if you deem it necessary. It is best to be consistent with your deviation across multiple elements. 

Be sure to only override methods defined in `HTMLElement` when necessary or desired. Examples of this could be `blur()` or `focus()`.

### Events
The `dispatchEvent` method allows us fire an instance of `CustomEvent`. Here it can make sense to prefix event names defined in the HTML specification. For example `org-textfield` might fire an event called `org-change`. This allows to tell custom element events and builtin events apart. 

You should not need to fire builtin events from your custom element. If you do, using a workaround like calling `click()` method on it might be easier and should also do the trick.   

## Slots
Slots can be useful for displaying user content. 

### Elements with one or more slots
If your element contains one slot or more (or you are not sure if you will have more than one slot) it is best practice to name your slots right away even if you only have one to start with. It is painful having to refactor your code when you introduce a second slot. 

In a language like Typescript it is a good idea to introduce a string enum which holds all your slots. This helps because from the name of the enum you can get code completion for all slots defined in that element. It also helps with refactoring later.

Example:
````typescript
export enum SampleElementSlots {
  CONTENT = 'content',
  ACTIONS = 'actions'
}
````
In your template:
````typescript
render() {
  return html`<sample-element>
      <p slot="${SampleElementSlots.CONTENT}">Hello world</p>
      <div slot="${SampleElementsSlots.ACTIONS}">
        <button>Click me!</button>
      </div>
    </sample-element>`;
}
````

### Elements that use a slot to display `textContent` 
If you have an element that as slotted content should only ever output text content you can achieve this like so:

````typescript 
@internalProperty()
protected text: string = '';

private updateSlottedContent(target) {
  this.text = target.assignedNodes().map((n) => n.textContent).join('');
}

render() {
  return html`<div class="fancy-styling">${this.text}</div>
    <div hidden>
      <slot @slotchange=${(event) => {this.updateSlottedContent(event.target);}}></slot>
    </div>`;
}
````
### Elements with only a single slot for user content
For some elements you want to allow to slot user content not only the text. If you are using a single slot you can emulate an API similar to that of builtin elements: 

````typescript 
@customElement('org-button')
export type OrgButton extends LitElement {

  render() {
    return html`<div class="button">
                  <slot></slot>
                </div>`;
  }
}
````
This example element could be used similarly to a builtin `button` element:

````html
<org-button><span class="icon"></span>Click me!</org-button>
````
Beware that this technique only works if you have a single slot.

### Styling slotted content
The pseudo selector `::slotted` can be used to style slotted content. Consider the following example: 
````typescript 
@customElement('org-button')
export type OrgButton extends LitElement {

  render() {
    return html`<div class="button">
                  <slot></slot>
                </div>`;
  }
}
````
Here all slotted content that is a p tag should have a font weight of bold:

````css
::slotted(p) {
  font-weight: bold;
}
````
More information on the slotted pseudo selector can be found at https://developer.mozilla.org/en-US/docs/Web/CSS/::slotted.

## Try to limit  direct DOM usage scenarios
While the DOM represents our markup and therefore our templates it presents us with certain challenges:

* Instances of elements can only be accessed once they have been rendered and added to the DOM
* Attribute values can only be strings. Any other types that you might want to use need to be serialized and deserialized in order to be passed through an attribute.

`lit-html` allows us to use property bindings in order to bind to element properties directly. This has multiple advantages:

* the binding value does not need to be serialized or deserialized
* the binding becomes "active" once the element is added to the DOM

Try to expose as much of your API as properties or attributes (if it is boolean or string attribute) to limit the scenarios where you need to query for a specific DOM element in order to achieve something. 

However, there are certainly scenarios where this is unavoidable for example if you want to invoke a method of an element or query attributes or properties that have not been exposed through an event.

## Allowing customization of your elements by developers
Shadow DOM protects your element styling against mutations from the outside. In some scenarios it might be useful to allow for some customization when using your elements though. In this case the specification allows you to drill a little hole into your Shadow DOM by defining some values in your styles as CSS variables. Developers can then define them at the host element.

For example in your component styles you might have:
````css
.button-primary {
  background-color: var(--primary-color);
  color: var(--text-color);
}
````
Users of your elements can then define these variables like so:
````css
:root {
  --primary-color: blue;
  --text-color: white;
}
````
More information on using CSS variables can be found here: https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties.




You can respond to user events on your component like double-clicking, hovering,
and key presses through event handlers. Simply implement the name of the event
you want to respond to as a method on your component.

For example, imagine we have a template like this:

```hbs
{{#double-clickable}}
  This is a double clickable area!
{{/double-clickable}}
```

Let's implement `double-clickable` such that when it is
clicked, an alert is displayed:

```app/components/double-clickable.js
import Ember from 'ember';

export default Ember.Component.extend({
  doubleClick() {
    alert("DoubleClickableComponent was clicked!");
  }
});
```

Browser events may bubble up the DOM which potentially target parent component(s)
in succession. To enable bubbling `return true;` from the event handler method
in your component.

```app/components/double-clickable.js
import Ember from 'ember';

export default Ember.Component.extend({
  doubleClick() {
    Ember.Logger.info("DoubleClickableComponent was clicked!");
    return true;
  }
});
```

See the list of event names at the end of this page. Any event can be defined
as an event handler in your component.

## Sending Event Data

In some cases your component's event handler may need to make use of the [data object](https://developer.mozilla.org/en-US/docs/Web/API/Event) associated with the event,
perhaps to support various draggable behaviors.
For example, in a component designed to have other elements dragged onto it,
we may need to send an `id` extracted from the drop event out to the javascript context of its parent template when that event is received in order to identify the dragged element.

To facilitate this, when we insert the component into its containing template,
we will pass the component an external action that will be used within the component to send the `id`:

```hbs
{{drop-target action=(action "didDrop")}}
```

Passing actions to components like this is discussed in detail in the guide [Triggering Changes with Events](https://guides.emberjs.com/v2.9.0/components/triggering-changes-with-actions/). 
Within the component the external action `didDrop` is now accessible as the `action` property and this action is designed to take our `id` parameter as an argument.

To send the `id`, we will call that action from the component's `drop` event handler.
To access the event object within the handler so that we can extract the `id`,
we simply include the event in the handler's function signature.
If you need to, you may also stop events from bubbling, by using `return false;`.

```app/components/drop-target.js
import Ember from 'ember';

export default Ember.Component.extend({
  attributeBindings: ['draggable'],
  draggable: 'true',

  dragOver() {
    return false;
  },

  drop(event) {
    let id = event.dataTransfer.getData('text/data');
    this.get('action')(id);
  }
});
```

## Event Names

The event handling examples described above respond to one set of events.
The names of the built-in events are listed below. Custom events can be
registered by using [Ember.Application.customEvents][customEvents].

Touch events:

* `touchStart`
* `touchMove`
* `touchEnd`
* `touchCancel`

Keyboard events

* `keyDown`
* `keyUp`
* `keyPress`

Mouse events

* `mouseDown`
* `mouseUp`
* `contextMenu`
* `click`
* `doubleClick`
* `mouseMove`
* `focusIn`
* `focusOut`
* `mouseEnter`
* `mouseLeave`

Form events:

* `submit`
* `change`
* `focusIn`
* `focusOut`
* `input`

HTML5 drag and drop events:

* `dragStart`
* `drag`
* `dragEnter`
* `dragLeave`
* `dragOver`
* `dragEnd`
* `drop`

[customEvents]: http://emberjs.com/api/classes/Ember.Application.html#property_customEvents

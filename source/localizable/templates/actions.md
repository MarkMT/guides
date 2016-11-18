Your app will often need a way to let users interact with controls that change application state.
For example, imagine that you have a template that shows a blog title,
and supports expanding the post to show the body when the user clicks the title.

Creating such a feature involves defining two things:
a behavior or *action* to be executed (showing the post) and a specific interaction event that will trigger that behavior (clicking the title).
In Ember, actions are defined by functions accessible within the execution context of the template (a controller or component),
while the [`action`](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_action) helper is used within the template to specify how those functions will be triggered.

There are two ways the `action` helper can be used.
First, if you add `{{action}}` to any HTML DOM element, when a user clicks the element,
the named function following the `action` keyword will be called by the template's corresponding component or controller.

```app/templates/components/single-post.hbs
<h3><button {{action "toggleBody"}}>{{title}}</button></h3>
{{#if isShowingBody}}
  <p>{{{body}}}</p>
{{/if}}
```

In the component or controller, that function is defined within the `actions` hook:

```app/components/single-post.js
import Ember from 'ember';

export default Ember.Component.extend({
  actions: {
    toggleBody() {
      this.toggleProperty('isShowingBody');
    }
  }
});
```

The use of the `action` helper in the way shown here is referred to as being in the *element space*.
We will discuss an alternative usage later on this page under [Closure Actions](/templates/actions/#toc_closure-actions).

You should use the <code>camelCased</code> event names, so two-word names like `keypress`
become `keyPress`.

## Action Parameters

You can optionally pass arguments to the action handler. Any values
passed to the `{{action}}` helper after the action name will be passed to
the handler as arguments.

For example, if the `post` argument was passed:

```handlebars
<p><button {{action "select" post}}>✓</button> {{post.title}}</p>
```

The `select` action handler would be called with a single argument
containing the post model:

```app/components/single-post.js
import Ember from 'ember';

export default Ember.Component.extend({
  actions: {
    select(post) {
      console.log(post.get('title'));
    }
  }
});
```

## Specifying the Type of Event

By default, when the `action` helper is used in the element space,
it listens for click events and triggers the action when the user clicks on the element.
However, you can specify an alternative event by using the `on` option.

```handlebars
<p>
  <button {{action "select" post on="mouseUp"}}>✓</button>
  {{post.title}}
</p>
```

## Allowing Modifier Keys

By default, when used in the element space,
the `{{action}}` helper will ignore click events with pressed modifier keys.
You can supply an `allowedKeys` option to specify which keys should not be ignored.

```handlebars
<button {{action "anActionName" allowedKeys="alt"}}>
  click me
</button>
```

This way the `{{action}}` will fire when clicking with the alt key
pressed down.

## Allowing Default Browser Action

By default, when used in the element space,
the `{{action}}` helper prevents the default browser action of the DOM event.
If you want to allow the browser action, you can stop Ember from preventing it.

For example, if you have a normal link tag and want the link to bring the user
to another page in addition to triggering an ember action when clicked, you can
use `preventDefault=false`:

```handlebars
<a href="newPage.htm" {{action "logClick" preventDefault=false}}>Go</a>
```

With `preventDefault=false` omitted, if the user clicked on the link, Ember.js
will trigger the action, but the user will remain on the current page.

With `preventDefault=false` present, if the user clicked on the link, Ember.js
will trigger the action *and* the user will be directed to the new page.

## Closure Actions

Instead of using the `action` helper in the element space context that we have described so far,
an alternative way of using this helper is shown in the following example.

```handlebars
<label>What's your favorite band?</label>
<input type="text" value={{favoriteBand}} onblur={{action "bandDidChange"}} />
```

The effect of this is to register the action `bandDidChange` as the handler for the `onblur` event on the `<input>` element shown.
The usage of the `action` helper shown here is described as being in the *attribute context* and the action produced in this way is referred to as a *closure action*.

There are two important differences about this usage compared with the element space context.
First, in the element space the `action` helper *itself* registers an event handler for the element it is attached to.
In contrast, here the helper simply returns the function that will be called when the event is triggered;
it is the explicit *assignment* of that function to the on-event attribute (in this case `onblur`) that registers the function as the handler for that event.

To be a little more specific, the `action` helper used in this context creates a *closure*,
binding the named function to any arguments that follow the function name.
To illustrate, if the `favoriteband` input in the previous example were one of a group of several different profile settings,
we might choose to use a more generic action and pass the name of the field to be updated as an argument of the action handler:

```handlebars
<input type="text" value={{favoriteBand}} onblur={{action "updateSetting" "favoriteBand"}} />
```

Here, the `action` helper binds the argument `"favoriteBand"` to the function `updateSetting` so that when the `onblur` event occurs,
the function will receive the setting name as its first argument:

```js
actions: {
  updateSetting(name) {
    // Change the named setting...
  }
}
```

The second important distinction with closure actions is that as well as function arguments specified explicitly using the `action` helper,
the associated browser event object is also passed to the action handler *automatically*.
Simply by including that argument in the function signature,
we gain access to the attributes of the event object within the function.
With a template containing the input element shown previously, the following handler definition will therefore receive both the setting name and the event object as arguments,
in this case printing the value of the user's input:

```js
actions: {
  updateSetting(name, event) {
    console.log(event.target.value);
    //...
  }
}
```

## Modifying the Action's First Parameter

When using the `action` helper to create a closure action, if a `value` option is specified,
its value will be considered a property path that will be read off the first parameter of the action.
This comes in handy with event listeners and enables us to work with one-way bindings.

To illustrate, we'll return to the earlier example using the `bandDidChange` action, but now with the `value` option:

```handlebars
<input type="text" value={{favoriteBand}} onblur={{action "bandDidChange" value="target.value"}} />
```

Without the `value` option,
the action handler would automatically receive the event object as its first argument (first since in this case there is no explicit argument provided to the action helper).
However with the `value` option specified as shown, the `target.value` property is extracted from the event and passed to the action instead,
removing the need for the action to extract the required data itself:

```js
actions: {
  bandDidChange(newValue) {
    console.log(newValue);
    //...
  }
}
```

## Currying Action Arguments

In the previous example, we chose to use the action `bandDidChange` without an explicit argument, since this meant that the event object would be the action's first argument,
allowing us to illustrate how the `value` option can be used to drill in to that object and extract the specific data we need.

However,
what if we want to be able to extract the user input from the event object before it is passed to the action in a case where there _is_ an explicit argument?
Ordinarily, in this case the event object would be passed to the action as its second argument,
as in the example we saw earlier:

```js
actions: {
  updateSetting(name, event) {
    //...
  }
}
```

However in this case, we can make use of the fact that a closure action's arguments *curry*, meaning that when we provide the first, explicit argument to the `action` helper, `{{action "updateSetting" "favoriteBand"}}`,
the helper returns a function that takes the remaining `event` argument as its *first* argument.
Executing that function is equivalent to invoking the original action function with both arguments.

This leads to the possibility of nesting `action` helpers as shown here:

```handlebars
<input type="text" value={{favoriteBand}} onblur={{action (action "updateSetting" "favoriteBand") value="target.value"}} />
```
The inner `action` returns a function that takes the event object as its single argument, but since we've used the `value` option on the outer `action`,
the `target.value` property will automatically be read off that argument first.
Because of this, we can modify the action definition to take the value of the user's input directly:

```js
actions: {
  updateSetting(name, value) {
    console.log(value)
    //...
  }
}
```

## Attaching Actions to Non-Clickable Elements

Note that actions may be attached to any element of the DOM, but not all
respond to the `click` event. For example, if an action is attached to an `a`
link without an `href` attribute, or to a `div`, some browsers won't execute
the associated function. If it's really needed to define actions over such
elements, a CSS workaround exists to make them clickable, `cursor: pointer`.
For example:

```css
[data-ember-action]:not(:disabled) {
  cursor: pointer;
}
```

Keep in mind that even with this workaround in place, the `click` event will
not automatically trigger via keyboard driven `click` equivalents (such as
the `enter` key when focused). Browsers will trigger this on clickable
elements only by default. This also doesn't make an element accessible to
users of assistive technology. You will need to add additional things like
`role` and/or `tabindex` to make this accessible for your users.

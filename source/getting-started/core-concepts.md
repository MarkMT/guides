To get started with Ember.js, there are a few core concepts you
should understand.

## Client-Side Page Composition
Like any web application, the content of an Ember application is rendered by a browser based on a Document Object Model representation conventionally described by HTML and CSS, with in-page behavior defined by Javascript. However unlike traditional server-based web applications, in Ember an HTML description of your application's content is not constructed in the server and sent to browser, ready to be rendered. Instead the HTML file loaded from the server is essentially just a container for the detailed content, which is constructed dynamically within the browser using Javascript. In this approach, the files that define the application (HTML, CSS, JS) are loaded just once, while subsequent interactions with the server are used to retrieve and update the data used by the application.

![Figure showing browser loading skeleton HTML and CSS/JS and generating app content dynamically](../../images/getting-started/core-concepts/Fig_1.jpg)

## Templates Separate Static and Dynamic Content
Like most web sites, an Ember application consists of both static content, often defining its layout and navigational structure, and dynamic content which changes both over time and depending on user selections etc. In Ember, as in many frameworks, this is achieved by expressing content in the form of *templates* that describe static content using HTML, with additional embedded expressions that define the dynamic content.

In Ember, templates are written in the HTMLBars language. An HTMLBars template can include expressions like `{{title}}` or `{{author}}` that refer to Javascript variables defined separately that contain the content to be inserted, and also control structures such as:

`{{if isAdmin}}30 people have viewed your blog today.{{/if}}`. 

The Ember framework does the work of inserting the dynamic content into the templates to create the actual application. And if anything causes the variables that define that content to change after it has initially been rendered (e.g. as a result of user input), Ember will automatically update the on-screen content to reflect that.

![Figure showing template loading dynamic data to create rendered page](../../images/getting-started/core-concepts/Fig_2.jpg)

Also note that the template files themselves are not sent to the browser. Rather, as part of the build process prior to deployment, these files are compiled to form, together with the Ember framework, the Javascript that will be loaded and executed by the browser to generate the application.

## Nested Templates Support Modularity and Reuse
Templates are not only used in Ember to define the relationship between static and dynamic content. They also play another important role in the way the overall visual layout of the user interface is designed: individual templates, represented by separate .hbs source files, can be nested within one another (As we'll discuss further below, there are a couple of different mechanisms that can be used to achieve this). Each of these templates also has access to its own Javascript scope and associated data, further supporting a modular approach to the overall design of the application.

![Figure showing nested templates accessing separate parts of app data](../../images/getting-started/core-concepts/Fig_3.jpg)

## A Conventional Web Model of Application Navigation
Another important feature of Ember is that it preserves the conventional web model of navigation in which different locations in your application are associated with distinct URLs. In a traditional website, different URLs are generally considered to represent different pages and navigating from one to another, say by clicking a menu link, usually involves loading a new HTML page from the server.

In an Ember application, we have to think a little differently, since navigating from one part of the application to another does not involve a conventional page load. Nonetheless, it is natural to think of different sets of nested templates that comprise the state of the user interface at different points in time as representing different 'locations' within the application. So by analogy, it makes sense to associate a URL with each of these states.

For example in one state we may see a list of titles of most recent blog posts. If we click on one of them, the state of the user interface may change to show the same list but also the full text of an individual post along with an edit link. If we click the edit link the user interface may still show the post but also a text field allowing you to edit the content of the post.

It's easy to imagine an interface like this being is composed of a number of nested templates. In this case the interface could consist of just a list template, or a list template containing a post template, or a list template containing a post template containing an edit template. Ember allows, and indeed encourages, you to associate each of these different states with a specific URL.

Why is this important? Because it means that if you navigate through the application to a particular 'location', you can easily share the corresponding URL with someone else knowing that they will be able to access the very same location, i.e. the same application state, just like you can with a traditional server-based application.

## Connecting Templates, URLs & Data -
## Routes, Models, Controllers & Components
We've talked in general terms above about the role that templates play in an Ember application and we've noted briefly the importance of the connections that exist between templates and URLs and between templates and data. Let's look a little more deeply at some of the implementation concepts that make these features possible.

In Ember, a *route* is an 'Ember object' defined in Javascript that serves to associate a specific URL (or in some cases a parameterized set of URLs) with (a) a specific template or set of templates to be rendered for that URL, and (b) any *persistent* data that is going to be made available for display by those templates. The mapping between the URL and a specific route is performed by another Ember Object called a *router*.

For simplicity, consider first the case of a URL that produces content described by just a single template, i.e. with no nesting<sup>[1](#footnote1)</sup>. Suppose the URL is `http://example.com/posts` and that the router maps that to a route called `posts`. By default this route will cause a template called `post.hbs` to be rendered.

The route also implements a method called `model()` that returns the persistent data available to the template. In general, the template has access to any data defined within another Ember object called a *controller*; this object has responsibility for invoking `model()` on the route and making the result available to the template via a property named `model`. By default, a controller will be loaded whether you explicitly define one or not (if only to provide the template with access to `model`), but you can also create one yourself explicitly and use it to define any additional variables required by the template.

Both the route and the controller also implement handler methods that are invoked in response to user actions. When an action occurs, Ember first looks for a handler with the required name within the controller. If it does not exist, the action will be sent instead to the route.

![Figure showing router, route, model, controller and template](../../images/getting-started/core-concepts/Fig_4.jpg)

For a URL representing a user interface consisting of nested templates, the router will map the URL to a *sequence* of routes, each associated with a single template and associated controller. Nesting is achieved through the use of an `{{outlet}}` expression within each template in the sequence, indicating where the next template in the sequence is to be inserted.

There is also another mechanism by which templates can be nested in Ember, one that does not involve routes (currently) or controllers. Instead it involves the use of *components*. A component consists of both a template and an associated Javascript module, very much like the template-controller combination shown above. However this approach differs in the way templates are inserted into one another and how those templates access data defined outside the component they are a part of:

1. A component's template is inserted into a containing template (which could be another component template or one associated with a route and controller in the way we described above) using a custom expression named after the specific template being inserted, e.g. `{{blog-post}}`<sup>[2](#footnote2)</sup>. Observe that this provides both the identity of the template to be inserted ('blog-post') and its location within the containing template. This stands in contrast to the case above where the identity of the template is specified by the route and the location is specified by the `{{outlet}}` expression.
2. Any external data used by the component, e.g. data from a containing template, including that from the `model()` hook of a route associated with a containing template, must be *passed* explicitly in the form of *attributes* attached to the component expression, e.g. `{{blog-post post=model}}`.

Another difference between components today and the controller-template combination shown previously is that components are not 'routable', meaning they cannot be associated with a route (and thus a URL) directly. In other words, a route cannot cause a component's template to be rendered directly in the same way that it can for one associated with a controller. A component cannot, therefore, be used as the 'top-level' definition of the application's user interface; it must be located within another template associated (either directly or indirectly) with a controller and route.

In a future release of Ember, however, components will become *routable* like controllers are now, and will ultimately replace controllers in the Ember architecture. In other words it will be possible to associate a route directly with a component to be inserted into an `{{outlet}}`. In this case, an `attributes()` hook defined in the route will provide the data to be passed to the component and this data will be available within the component via a predefined `attrs` property.

![Figure showing routable components](../../images/getting-started/core-concepts/Fig_5.jpg)

## Transparent Access to Persistent Data
In Ember the term *model* is used in two slightly different ways. As we noted above, routes implement a `model()` method in order to provide a controller, and indirectly a template, with access to data that the template will render. That data may consist of a single javascript object or an array of objects. Although the data can be defined by the `model()` method directly, e.g. as a Javascript object literal, usually -

1. the data is intended to be *persistent*, meaning that objects stored locally are actually copies of records stored somewhere other than the browser, and 
2. those records are each instances of a particular class that defines the attributes or properties the records possess.

To support this, in Ember a *store* object provides access to this data locally and transparently mediates access to the remote data via an *adapter*, while a set of *model classes* define the the specific attributes of the records accessible via the store. So individual classes define, for example, what a `post`, a `user` or a `line-item` etc are.

![Figure showing route access store accessing cloud via adapter](../../images/getting-started/core-concepts/Fig_6.jpg)

### Footnotes
<a name="footnote1">1</a>: In fact all templates are nested inside the default top-level 'application' template, so the situation we're describing is where there is just one such template.

<a name="footnote2">2</a>: In a future release of Ember, this syntax will be replaced using angle-brackets as the delimiters, so that the component will appear as a custom html element.

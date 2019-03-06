Components are an essential building block in Ember applications. They allow
developers to package presentation and behaviour into a single unit and give it
a name. You can think of them like defining your own custom HTML elements, like
a custom `<input>` or `<select>` tag that has its own behavior, values, and
events that you can hook into in your templates.

Like we mentioned in the section on [Templates](../../templates/), Components
can have both a template and a class definition:

```hbs {data-file=app/templates/components/hello-button.hbs}
<button onclick={{this.sayHello}}>
  {{@buttonText}}
</button>
```

```js {data-file=app/components/hello-button.js}
import Component from '@glimmer/component';
import action from '@ember/object';

export default class HelloButton extends Component {
  @action
  sayHello() {
    console.log('Hello, world!');
  }
}
```

And they can be used in other templates:

```hbs
<HelloButton @buttonText="Say Hello!"/>
```

The syntax for using a component is similar to standard HTML elements, but uses
`<CapitalCase>` for the name of the component instead of lower case. This
_invocation_, as we call it, for the `HelloButton` component results in the
following HTML output wherever it was used:

```html
<button>
  Say Hello!
</button>
```

And when we click on the button, it triggers its `onclick` handler, logging
"Hello, world!" to the console. We can reuse this component as many times as we
like, and we can even change the text via the value that we pass to it,
`@buttonText`, which is known as an _argument_:

```hbs
<HelloButton @buttonText="Say Hello!"/>
<HelloButton @buttonText="Di Hola!"/>
<HelloButton @buttonText="Dis Bonjour!"/>
```

Results in:

```html
<button>
  Say Hello!
</button>
<button>
  Di Hola!
</button>
<button>
  Dis Bonjour!
</button>
```

However, clicking on each button will still log the same "Hello, world!"
message, since the click handler is defined in the _class_ of the component, and
doesn't use any arguments. We'll talk more about arguments - and their
counterpart, _actions_ - later on.

Components can also have _blocks_, just like standard HTML elements. We could
update the `HelloButton` component to render its button text from its block
instead of the `@buttonText` argument:

```hbs {data-file=app/templates/components/hello-button.hbs}
<button onclick={{this.sayHello}}>
  {{yield}}
</button>
```

```hbs
<HelloButton>
  Say Hello!
</HelloButton>
```

This is the same as our first example, with the `HelloWorld` button component
_yielding_ to the block that was passed to it instead of being passed an
argument. We can put anything inside of that block, including text, HTML, and
other components. This is part of what makes components so powerful and
composable as a whole.

### Generating a Component

You can generate a component using the Ember CLI:

```bash
ember generate component blog-post
```

This generates a JavaScript file:

```javascript {data-file=src/ui/components/blog-post/component.js}
import Component from '@glimmer/component';

export default class BlogPostComponent extends Component {}
```

And a template file:

```handlebars {data-filename=src/ui/components/blog-post/template.hbs}
{{yield}}
```

We will update the template to greet the user:

```handlebars {data-filename=src/ui/components/blog-post/template.hbs}
<p>Hi user!</p>
```

If you call this component from a route template, you will see `<p>Hi user!</p>`
being rendered when you visit that route:

```handlebars {data-filename=src/ui/routes/application/template.hbs}
<BlogPost />
```

### Component Folder Structure

## Dynamically rendering a component

The
[`{{component}}`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/component?anchor=component)
helper can be used to defer the selection of a component to run time. The
`<MyComponent />` syntax always renders the same component, while using the
`{{component}}` helper allows choosing a component to render on the fly. This is
useful in cases where you want to interact with different external libraries
depending on the data. Using the `{{component}}` helper would allow you to keep
different logic well separated.

The first parameter of the helper is the name of a component to render, as a
string. So `{{component 'blog-post'}}` is the same as using `<BlogPost />`.

The real value of
[`{{component}}`](https://www.emberjs.com/api/ember/release/classes/Ember.Templates.helpers/methods/component?anchor=component)
comes from being able to dynamically pick the component being rendered. Below is
an example of using the helper as a means of choosing different components for
displaying different kinds of posts:

```handlebars {data-filename=src/ui/components/foo-component/template.hbs}
<h3>Hello from foo!</h3>
<p>{{this.post.body}}</p>
```

```handlebars {data-filename=src/ui/components/bar-component/template.hbs}
<h3>Hello from bar!</h3>
<div>{{this.post.author}}</div>
```

```handlebars {data-filename=src/ui/routes/index/template.hbs}
{{#each this.myPosts as |post|}}
  {{!-- either foo-component or bar-component --}}
  {{component post.postType post=post}}
{{/each}}
```

or

```handlebars {data-filename=src/ui/routes/index/template.hbs}
{{#each this.myPosts as |post|}}
  {{!-- either foo-component or bar-component --}}
  {{#let (component post.postType) as |Post|}}
    <Post @post={{post}} />
  {{/let}}
{{/each}}
```

When the parameter passed to `{{component}}` evaluates to `null` or `undefined`,
the helper renders nothing. When the parameter changes, the currently rendered
component is destroyed and the new component is created and brought in.

Picking different components to render in response to the data allows you to
have different template and behavior for each case. The `{{component}}` helper
is a powerful tool for improving code modularity.

## Template-only components

Components can also consist of _just_ a template definition (or in some cases,
just a class definition). Components with just a template are known as
Template-Only components, as well as presentational or functional components.
The major difference is that _unlike_ components with a class, Template-Only
components are _stateless_ - they are purely based on the _arguments_ that they
are passed. This makes them much easier to reason about, and very useful in many
circumstances.

What this means in practice is that using properties in the template
(`{{this.myProperty}}`) will result in an error. In a template-only component
you can only use values that were passed to the component, called named
arguments (`{{@myArgument}}`).

A small example would be a greeting component that receives the name of a friend
and greets them:

```handlebars {data-filename=src/ui/components/greeting/template.hbs}
<p>Hello {{@friend}}</p>
```

```handlebars {data-filename=src/ui/routes/application/template.hbs}
<Greeting />
```

We will learn more about properties and named arguments in the [displaying data
guide]().

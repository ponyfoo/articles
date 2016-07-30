Particularly, we'll be moving the server-side layout rendering out of `express-hbs` and into another React component, _the `<Layout />` component_, which will only be rendered on the server. The reason for that change is one of consistency -- instead of having to understand _(and deal with)_ two different templating languages in Handlebars and JSX, we'll only be dealing with JSX.

After making that change we'll be looking at a couple of [`react-side-effect`][1]-powered libraries. One of these will enable us to set the `document.title` declaratively in our views from anywhere in our JSX templates. The other allows us to define `<meta>` tags anywhere in the document as well.

> Let's get on with it!

[1]: https://github.com/gaearon/react-side-effect

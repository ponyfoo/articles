# Obey and Report

Recently, I came up with this idea where I laid out a library that _doesn't necessarily depend on Angular_, but which plays very nicely with it. The pattern basically dictates that the components **can only change their state through commands**, which can come from either its UI directly, or be _programatically invoked_. This allows me to provide a default UI for the component, but is flexible enough to let an Angular application take over and replace that UI layer with its own, which then simply dictates the library to obey the commands it's sent. **Reporting** happens whenever a public state property is updated, and the UI could use said property to update its _representation_ of the component's state.

![obey-report][1]

  [1]: https://i.imgur.com/64esjO6.png "Obey and Report Pattern"

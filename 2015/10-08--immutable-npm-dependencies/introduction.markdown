In theory, semantic versioning is a great way of keeping our packages up to date all the time, but in reality it's not all peaches and cream. If you or anyone in the dependency chain is relying on an undocumented feature _(we shouldn't do this, but it happens)_ and the package in question changes their internals, or inadvertently introduces a bug _(we shouldn't do this, but it happens)_, or mistakenly breaks their API _(we shouldn't do this, but it happens)_, you're screwed.

> Even if you follow semver strictly someone somewhere up the chain is going to use your code in a way that you'll break.

I wrote about this topic [a while back][1], and then again over the past few days on Twitter, but in this case I'm arguing that the current model is broken and we should pin dependencies for all the modules, or find a better way of making `npm` users `shrinkwrap` their "final" packages _(like CLI apps or web apps)_.

[1]: /articles/semver "Pragmatic Semantic Versioning on Pony Foo"

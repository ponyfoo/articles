Always make sure to thoroughly test the logic in your packages before unleashing them into the wild. If you happen to publish a _faulty image_ of your package, you **can overwrite** it using the `--force` flag:

```bash
$ npm publish --force
```
    
In addition to **.gitignore**, **.npmignore** files can come in handy, here is an example, again from the **jsn** repository on _GitHub_.

```
*.md
.DS_Store
.git*
.idea
Makefile
docs/
examples/
support/
test/
```

Spread the word!

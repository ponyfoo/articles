Angular 2 requires a bit of setup to get started. To avoid the headaches associated with setup, the Angular team came up with the [Angular CLI][cli]. The Angular 2 CLI makes it easy to create an application that just works out of the box.

Install the Angular 2 CLI globally:

```bash
npm install -g angular-cli
```

**Note:** The Angular team has decided to drop the 2 from the name. So, it is now called **Angular** instead of **Angular 2**. For the sake of this tutorial, I'll use Angular 2 to prevent confusion from developers just trying out the framework for the first time.

Use these commands to simply create your app and run it:

```bash
ng new myapp // creates a new app
ng generate // generates components, routes, services and pipes
ng serve // serves your application in the browser
```

In this tutorial, we'll avoid using the CLI and learn how to set up our development environment from scratch. Meanwhile if you are interested in using the CLI with [Cloudinary][cloudinary], check out this [great sample][cloudinary-angular]. Should we use SystemJS? Is Webpack the best option? How does the transpiling work? You'll get the answers to these questions as we get our hands dirty with the nitty-gritty of setting up and Angular 2 application.

[cli]: https://cli.angular.io/
[cloudinary]: http://cloudinary.com/?utm_source=ponyfoo&utm_medium=Sponsored_post_1&utm_content=Angular2
[cloudinary-angular]: https://github.com/cloudinary/cloudinary_angular/tree/angular_next/samples/angular-cli-sample

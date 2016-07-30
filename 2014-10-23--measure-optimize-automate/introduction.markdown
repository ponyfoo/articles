Pretty much every web performance engineer out there starts by telling you to **measure performance**, and they typically suggest you measure it by tracking your relevant business metrics _(such as sign-ups, purchases, or renewals)_, as a way to entice management and teach them that [#perfmatters][1]. Once you have an idea of the current state of your application, you can go over the performance report, fix the glaring issues, and measure again. The **iterative approach allows you to keep your application on a tight leash**, which helps you to quickly deploy improvements that make your site faster. Since you're measuring your performance, you can _gauge exactly how beneficial your changes were_.

Of course, all of this **needs to be automated** in order to be a feasible and sustained effort. In this article I'll explore our options when it comes to measuring, learning from those measurements, and automating them.

Since we're [not really concerned with a specific build tool][2], I'll default to showing examples from the command-line, so that you can run them using [npm run][3], and then provide links to well known plugins for the popular kids in the build block - [Grunt][4], [Gulp][5] and [Broccoli][6] - as well as short explanations on how to use those plugins.

[![Gulp, Grunt, Whatever][7]][2]

[1]: https://twitter.com/hashtag/perfmatters
[2]: /articles/gulp-grunt-whatever
[3]: http://substack.net/task_automation_with_npm_run
[4]: http://gruntjs.com/
[5]: http://gulpjs.com/
[6]: https://github.com/broccolijs/broccoli
[7]: https://i.imgur.com/rVlIUsC.jpg "Gulp, Grunt, Whatever"

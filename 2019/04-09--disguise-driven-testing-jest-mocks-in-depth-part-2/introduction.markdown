Mocks are a great way of preventing AJAX calls in tests, but they can also help you isolate side effects and impurities that can create complicated tests.

As you [learned in Part 1][pt1], mocks are a great way to handle external data or any data that is likely to change. Mocking external data will likely be your most common use case and for a good reason. You want your tests to stick as closely to your code as possible this includes all dependencies. Still, there are times when a dependency creates a testing specific problem. In other words, the code works as you would want it to in production, but in a testing environment can make consistent, predictable tests hard or nearly impossible.

Some of your third party code may have side effects or some form of impurity that will complicate your testing. Maybe the code has certain expectations of the DOM. Maybe it will change slightly depending on the order of operations. In all cases, the code is out of your control, but you need it to be predictable.

[pt1]: /articles/disguise-driven-testing-jest-mocks-in-depth "Disguise-Driven Testing: Jest Mocks in Depth — Part 1 on Pony Foo"

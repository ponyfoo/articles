# Traditional Paging

Something along these lines is what traditionally is done in order to _implement_ a paged list:

- First of all, the list is **sorted**
- The first page of the list is provided
- A **pager**, or list of pages is presented
- When the user **clicks** on a page, a new page is loaded

![traditional-pager.png][1]

### What's so wrong with paging?

This implementation of a paged list is rather **rudimentary**. The major flaw of traditional paged lists, is the _typically_ **incorrect assumption** that the user really cares about _a particular page_, or which particular page they are _looking at_.

Have you ever wanted to go to a **particular** page in a list on a website?  
How about, for a reason _other than **"because I figure the result should be more or less around... here"**_?

> Page numbers are, in the _vast majority_ of cases, an **implementation detail**. And they should be treated as such.

Another, and often _overlooked_, issue with traditional pagers, is the fact that you have to actually **click** on a button _every time_ you want to see another page of results. If for some reason you are sifting through a list, clicking every time you want to see more results becomes pretty obnoxious.

To make matters _worse_, paging usually performs a **full page reload**. This, coupled with clicking, sums up a rather _frustrating experience_.

In summary, traditional paging is bad because:

- It assumes page indexes are _somehow relevant_
- It _requires user action_ to navigate
- It takes _too long_

What's a good alternative?

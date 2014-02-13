# Paging in the Wilderness #

Traditionally, lists on websites have been paged to allow data to be _chunked_ rather than sent out all at once, when the user might not need all of it _right away_, improving both _overall performance_ and _perceived speed_, both of which are very important to **web usability**.

# Traditional Paging #

Something along these lines is what traditionally is done in order to _implement_ a paged list:

- First of all, the list is **sorted**
- The first page of the list is provided
- A **pager**, or list of pages is presented
- When the user **clicks** on a page, a new page is loaded

![traditional-pager.png][1]

### What's so wrong with paging? ###

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

### An alternative approach ###

> It's easy to complain about something existent and proven, but how to _make it better_?

> Pretty easy, actually. Getting rid of all the issues I ranted about.

- Don't care about _indexes_? **Remove them** altogether.  
- **Clicks** have a _negative impact on UX_? Find a more **natural** way to do paging.  
- **Reloads** are _slow and inefficient_? Go **AJAX**.

## Unobtrusive Paging ##

Instead of a pager, we'll use a footer at the end of our page, and when the user scrolls past the footer, the next page will be loaded, through an AJAX request and some JavaScript code.

![unobtrusive-pager.png][2]

Now **all you have to do** is:

 1. Go to step two if the list is depleted, otherwise skip to step three
 2. Use a **"No more content"** footer when the list is depleted
 3. Use the paging footer when the list has more items
 4. Bind `click` and `scroll` events to the paging footer
 5. Append new contents, go _back to step one_

I won't go into the implementation of all of these steps, I'll leave that _up to you_. However, here's a snippet laying out an example of how you could implement the _scrolling_ functionality:
 
 ```js
var win = $(window),
    pager = $('.pager');
    
function more(){
    win.off('scroll.paging');
    pager.off('click.paging');
    pagingEvent(pager, data);
}

win.on('scroll.paging', function(){
    var allowance = 80,
        target = pager.position().top + pager.height() - allowance,
        y = win.scrollTop() + win.height();

    if (y > target){
        more();
    }
});

pager.on('click.paging', more);
```

Make sure to set an `allowance` that lets your pager _scroll into view_, but doesn't force the user to go _all the way down_ to the bottom of the page.

After that, you should implement `pagingEvent` to fetch the **next page** and append it to what you currently have, remove the existing pager, and then figure out whether you are going to display a **"No more results"** element, or **another pager** (you could set this up _recursively_).

This provides a **frictionless** experience, where the user can sift through your paged list _just by scrolling down_ the page, or using  <kbd>PgDown</kbd>.

> It also avoids the _need for an unnecessary clickfest_, and **feels more natural** overall, which is **the essence** of designing an _**enjoyable user experience**_.

So there you have it, sometimes it's nice to look at everyday things we take for _granted_, and review how they could be **improved** to deliver a better product.

  [1]: http://i.imgur.com/7sShhiX.png "The most common implementation of paging"
  [2]: http://i.imgur.com/61NF6fE.png "The unobtrusive way"

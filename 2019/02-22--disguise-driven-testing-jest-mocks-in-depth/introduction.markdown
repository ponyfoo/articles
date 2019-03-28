Consider an example. Let’s say you needed to write some code that checks to see if a given date is two weeks from today. If they date is far enough away, you get an empty string. If the date is less than two weeks away, you get an error message:

```js
import moment from 'moment';

export const message = 'Date must be two weeks from now'

export function isAtLeastTwoWeeks(date) {
  const isLater = moment(date).isAfter(moment().add(2, 'weeks'));
  return isLater ? '' : message;
}
```

That’s pretty simple code. But how do you write a test for it? Things can get tricky fast.

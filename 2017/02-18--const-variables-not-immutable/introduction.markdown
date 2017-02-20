The following example shows that even though the `people` reference couldn't be changed, the array itself can indeed be modified. If the array were immutable, this wouldn't be possible.

```js
const people = ['Tesla', 'Musk']
people.push('Berners-Lee')
console.log(people)
// <- ['Tesla', 'Musk', 'Berners-Lee']
```

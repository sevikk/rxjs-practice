## forkJoin, zip, combineLatest, withLatestFrom

[**DEMO**](https://stackblitz.com/edit/scotch-rxjs-combine?file=src/app/app.component.ts)

**zip**

```js
// 3. We are ready to start printing shirt...
zip(color$, logo$)
    .subscribe(([color, logo]) => console.log(`${color} shirt with ${logo}`));
```

For those of you who are not familar with JavaScript ES6/ES2015 {destructuring assignment}(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), you might find the syntax in subscribe **[color, logo]** a little bit odd.

When we zip **color$** and **logo$**, we expect to receive an array of 2 items during subscribe, first item is color and second is logo (follow their orders in zip function).

The traditional way of writing it would be ```js.subscribe((data) => console.log(${data[0]} shirt with ${data[1]}))```. As you can see, it's not very obvious that **data[0]** is color.

ES6 allows us to unpack the value from arrays. Therefore, we unpack data into **[color, logo]** straight away. More readable right?
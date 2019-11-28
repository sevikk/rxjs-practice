## Higher-Order Observable

There is a simple example of observable
Copy and paste it in https://rxviz.com/

```js
const { interval, from } = Rx;
const { take, map, tap } = RxOperators;

interval(1000).pipe(
    map(number => from(
        fetch(`https://jsonplaceholder.typicode.com/todos/${number + 1}`)
              .then(response => response.json())
        )
    ),
    // I expected `event` to be an HTTP response body, not another Observable
    tap(event => console.log(event))
)
```

RxJS provides of and from to convert single values, arrays, objects that emit events, and promises into observables. If your application converts something into an observable from inside the map operator, the next operator in your pipe will receive an observable, and you'll have a stream of streams. That means you outer observable has become a higher-order observable.

You can do this without using the of or from functions directly because libraries you rely on might use them. For example, Angular's http service uses of to convert an HttpRequest into an observable. That means making HTTP request within an observable's stream turns it into a higher-order observable.

Operators like groupBy turn one stream into many based on the result of a function. This is helpful when categorizing events to treat them in different ways. It also creates a higher-order observable - the initial stream will now emit more streams of categorized events rather than its values.

**Simple example of Highee-Order Observable**

```js
const { interval } = Rx;
const { take, map } = RxOperators;

interval(1000).pipe(
    // use `interval` inside `pipe` and `map` to make a stream inside a stream
    map(() => interval(500))
)
```
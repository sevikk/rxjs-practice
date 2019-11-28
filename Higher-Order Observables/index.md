## There is a aresult of http request 

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
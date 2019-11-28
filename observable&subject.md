In stream programming there are two main interfaces: Observable and Observer.

**Observable** is for the consumer, it can be transformed and subscribed:

```observable.map(x => ...).filter(x => ...).subscribe(x => ...)```

**Observer** is the interface which is used to feed an observable source:

```observer.next(newItem)```

We can create new Observable with an Observer:

```js
const observable = Observable.create(observer => { 
    observer.next('first'); 
    observer.next('second'); 
    ... 
});
observable.map(x => ...).filter(x => ...).subscribe(x => ...)
```

Or, we can use a **Subject** which implements both the **Observable** and the **Observer** interfaces:

```js
const source = new Subject();
source.map(x => ...).filter(x => ...).subscribe(x => ...)
source.next('first')
source.next('second')
```
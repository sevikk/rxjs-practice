## Subjecs

**BehaviorSubject**

One of the variants of the Subject is the BehaviorSubject. The BehaviorSubject has the characteristic that it stores the “current” value. This means that you can always directly get the last emitted value from the BehaviorSubject.

here are two ways to get this last emited value. You can either get the value by accessing the .value property on the BehaviorSubject or you can subscribe to it. If you subscribe to it, the BehaviorSubject will directly emit the current value to the subscriber. Even if the subscriber subscribes much later than the value was stored. See the example below:

```js
import * as Rx from "rxjs";

const subject = new Rx.BehaviorSubject();

// subscriber 1
subject.subscribe((data) => {
    console.log('Subscriber A:', data);
});

subject.next(Math.random());
subject.next(Math.random());

// subscriber 2
subject.subscribe((data) => {
    console.log('Subscriber B:', data);
});

subject.next(Math.random());

console.log(subject.value)

// output
// Subscriber A: 0.24957144215097515
// Subscriber A: 0.8751123892486292
// Subscriber B: 0.8751123892486292
// Subscriber A: 0.1901322109907977
// Subscriber B: 0.1901322109907977
// 0.1901322109907977
```

There are a few things happening here:

* We first create a subject and subscribe to that with Subscriber A. The Subject then emits it’s value and Subscriber A will log the random number.
* The subject emits it’s next value. Subscriber A will log this again
* Subscriber B starts with subscribing to the subject. Since the subject is a BehaviorSubject the new subscriber will automatically receive the last stored value and log this.
* The subject emits a new value again. Now both subscribers will receive the values and log them
* Last we log the current Subjects value by simply accessing the .value property. This is quite nice as it’s synchronous. You don’t have to call subscribe to get the value.

Last but not least, you can create BehaviorSubjects with a start value. When creating Observables this can be quite hard. With BehaviorSubjects this is as easy as passing along an initial value. See the example below:

```js
import * as Rx from "rxjs";

const subject = new Rx.BehaviorSubject(Math.random());

// subscriber 1
subject.subscribe((data) => {
    console.log('Subscriber A:', data);
});

// output
// Subscriber A: 0.24957144215097515
```


**ReplaySubject**

The ReplaySubject is comparable to the BehaviorSubject in the way that it can send “old” values to new subscribers. It however has the extra characteristic that it can record a part of the observable execution and therefore store multiple old values and “replay” them to new subscribers.

When creating the ReplaySubject you can specify how much values you want to store and for how long you want to store them. In other words you can specify: “I want to store the last 5 values, that have been executed in the last second prior to a new subscription”. See example code below:

```js
import * as Rx from "rxjs";

const subject = new Rx.ReplaySubject(2);

// subscriber 1
subject.subscribe((data) => {
    console.log('Subscriber A:', data);
});

subject.next(Math.random())
subject.next(Math.random())
subject.next(Math.random())

// subscriber 2
subject.subscribe((data) => {
    console.log('Subscriber B:', data);
});

subject.next(Math.random());

// Subscriber A: 0.3541746356538569
// Subscriber A: 0.12137498878080955
// Subscriber A: 0.531935186034298
// Subscriber B: 0.12137498878080955
// Subscriber B: 0.531935186034298
// Subscriber A: 0.6664809293975393
// Subscriber B: 0.6664809293975393
```

There are a few things happening here:

* We create a ReplaySubject and specify that we only want to store the last 2 values
* We start subscribing to the Subject with Subscriber A
* We execute three new values trough the subject. Subscriber A will log all three.
* Now comes the magic of the ReplaySubject. We start subscribing with Subscriber B. Since we told the ReplaySubject to store 2 values, it will directly emit those last values to Subscriber B and Subscriber B will log those.
* Subject emits another value. This time both Subscriber A and Subscriber B just log that value.



**AsyncSubject**

While the BehaviorSubject and ReplaySubject both store values, the AsyncSubject works a bit different. The AsyncSubject is aSubject variant where only the last value of the Observable execution is sent to its subscribers, and only when the execution completes. See the example code below:

```js
import * as Rx from "rxjs";

const subject = new Rx.AsyncSubject();

// subscriber 1
subject.subscribe((data) => {
    console.log('Subscriber A:', data);
});

subject.next(Math.random())
subject.next(Math.random())
subject.next(Math.random())

// subscriber 2
subject.subscribe((data) => {
    console.log('Subscriber B:', data);
});

subject.next(Math.random());
subject.complete();

// Subscriber A: 0.4447275989704571
// Subscriber B: 0.4447275989704571
```

This time there’s not a lot happening. But let’s go over the steps:

* We create the AsyncSubject
* We subscribe to the Subject with Subscriber A
* The Subject emits 3 values, still nothing hapening
* We subscribe to the subject with Subscriber B
* The Subject emits a new value, still nothing happening
* The Subject completes. Now the values are emitted to the subscribers which both log the value.

The BehaviorSubject, ReplaySubject and AsyncSubject can still be used to multicast just like you would with a normal Subject. They do however have additional characteristics that are very handy in different scenario’s.
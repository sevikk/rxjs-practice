## forkJoin, zip, combineLatest, withLatestFrom

[**DEMO**](https://stackblitz.com/edit/scotch-rxjs-combine?file=src/app/app.component.ts)

**zip**

```js
// 3. We are ready to start printing shirt...
zip(color$, logo$)
    .subscribe(([color, logo]) => console.log(`${color} shirt with ${logo}`));
```

For those of you who are not familar with JavaScript ES6/ES2015 [destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), you might find the syntax in subscribe **[color, logo]** a little bit odd.

When we zip **color$** and **logo$**, we expect to receive an array of 2 items during subscribe, first item is color and second is logo (follow their orders in zip function).

The traditional way of writing it would be ```.subscribe((data) => console.log(${data[0]} shirt with ${data[1]}))```. As you can see, it's not very obvious that **data[0]** is color.

ES6 allows us to unpack the value from arrays. Therefore, we unpack data into **[color, logo]** straight away. More readable right?

**How does zip work?**

Again, zip operator is the love birds operator. In our case, color will wait for logo whenever there are new value (vice versa). Both values must change then only the log gets triggered.

```zip``` operator can accept more than 2 observables - no matter how many observables, they must all wait for each other, no man left behind!

**combineLatest**

I call combineLatest operator the go dutch operator. They are independent and doesn't wait for each other, they take care of themselves.

```js
// 3. We are ready to start printing shirt...
combineLatest(color$, logo$)
    .subscribe(([color, logo]) => console.log(`${color} shirt with ${logo}`));
```

**How does combineLatest work?**

As mentioned, combineLatest is the go dutch operator - once they meet their mates one time, they will wait for no man. In our case, first function is triggered after both color and logo values change. There onwards, either color or logo value changed will trigger the log.

**withLatestFrom**

I call withLatestFrom operator the master slave operator. At first, master must meet the slave. After that, the master will take the lead, giving command.

```js
// 3. We are ready to start printing shirt...
color$.pipe(withLatestFrom(logo$))
    .subscribe(([color, logo]) => console.log(`${color} shirt with ${logo}`));
```

**How does withLatestFrom work?**

Can you guess who is the master and who is the slave in our case?

You guessed it! **color** is the master while **logo** is the slave. At first (only once), **color(master)** will look for **logo(slave)**. Once the **logo(slave)** has responded, **color(master)** will take the lead. Log will get triggered whenever the next **color(master)** value is changed. The **logo(slave)** value changes will not trigger the console log.

**forkJoin**

Kind of final destination! I call forkJoin operator the final destination operator because they are very serious, they only commit once all parties are very sure that they are completely true, final destination of each other.

```js
// 3. We are ready to start printing shirt...
forkJoin(color$, logo$)
    .subscribe(([color, logo]) => console.log(`${color} shirt with ${logo}`));
```

**How does forkJoin work?**

**forkJoin** is the final destination operator! They are very serious to make sure each other are their final destination. In our code, both **color** and **logo** observables are not complete, we can keep pushing value by calling **.next** - that means they are not serious enough and thus they are not final destination of each other.

We need to complete both observables. Let's replace our setup code part 5 with the below:

```js
// 5. When the two persons(observables) ...
color$.complete();
logo$.complete();
```

here is more than one way to complete observable. There are operators that allow you to auto complete observable when conditions met, for example [```take```](https://rxjs-dev.firebaseapp.com/api/operators/take), [```takeUntil```](https://rxjs-dev.firebaseapp.com/api/operators/takeUntil), [```first```](https://rxjs-dev.firebaseapp.com/api/operators/first) etc.

Let's say, you only want to make 1 shirt, you only need to know the first **color** and **logo**, In this case, you don't care about the rest of the info that Ms. Color & Mr. Logo provide. You can make use of **take** or **first** operator to achieve auto complete observable once first **color** and **logo** emit.

Let's replace the setup code part 3 with the below code:
```js
// 3. We are ready to start printing shirt...
const firstColor$ = color$.pipe(take(1));
const firstLogo$ = logo$.pipe(first());

forkJoin(firstColor$, firstLogo$)
    .subscribe(([color, logo]) => console.log(`${color} shirt with ${logo}`));
```

You can remove all the code in part 5 as well, we don't need the two lines .complete() (as previous code) because take and first will auto complete the observable when the condition met.

*zip - the love birds, always work as a team, triggers only when all observables return new values
*combineLatest - the go dutch, start trigger once all observables return new values, then wait for no man, trigger every time when either observable return new value.
*withLatestFrom - the master slave, master first waits for slave, after that, action get triggered every time only when master return new value.
*forkJoin - the final destination, trigger once when all observables have completed.

**Which operator should I use?**
So I guess you can answer "which operator should I use?" better now. As a general rule of thumb - choose the one that works for you. In some cases, the outcome of using different operators might be the same (that's why people get confused on which one to use), it would be good to understand the intention of the operator & decide accordingly.

One of the most common use case of combination operators would be calling a **few apis, wait for all results return, then executing next logic**. Either forkJoin or zip will work and return same result because api calls are one-time only, auto-completed once result is returned (e.g. Angular httpClient.get)

However, by understanding the operators more, **forkJoin** might be more suitable in this case. It is because we "seriously" want to wait for all http responses to complete before proceed to the next step. **zip** is intended for observables with multiple emits. In our case, we expect only one emit for each http request. Therefore, I think **forkJoin** is more appropriate (oh well, either way, your code will run just fine & return the same result).
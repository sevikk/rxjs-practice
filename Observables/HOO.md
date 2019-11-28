## Higher-Order Observable

There is a simple example of observable.
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

**What does it mean?**

*The top line represents the first stream that was created by the outermost observable, interval(1000)
*Grey circles on that line represent new observables being created inside the map function. Each one was created by interval(500)
*Each line down the screen represents a new stream from one of those new observables
*Each new line across the screen with numbers represents the output of an observable created by interval(500)
*Each successive internal stream starts further to the right because it begins later.

**Flattening Higher-Order Observables into regular, First-Order Observables**

```js
const { interval, range, timer } = Rx;
const { take, map, groupBy, tap, flatMap, switchMap } = RxOperators;

const setsOfValues = [
  ["a", "b", "c", "d", "e", "f"],
  [1, 2, 3, 4, 5, 6],
  ["😀", "🐶", "🍏", "⚽️", "🚗", "⌚️"]
];

const threeStreamsOfThings$ = timer(0, 1800).pipe(
  take(3),
  map(outerNumber =>
    timer(0, 250).pipe(
      take(6),
      map(innerNumber => setsOfValues[outerNumber][innerNumber])
    )
  )
);

threeStreamsOfThings$
```

**Let's try flatMap**

flatMap is the easier operator to understand. When applied to one observable, it captures all the events emitted by nested observables and moves them up into the first observable's stream.

```js
const { interval, range, timer } = Rx;
const { take, map, groupBy, tap, flatMap, switchMap } = RxOperators;

const setsOfValues = [
  ["a", "b", "c", "d", "e", "f"],
  [1, 2, 3, 4, 5, 6],
  ["😀", "🐶", "🍏", "⚽️", "🚗", "⌚️"]
];

const threeStreamsOfThings$ = timer(0, 1800).pipe(
  take(3),
  map(outerNumber =>
    timer(0, 250).pipe(
      take(6),
      map(innerNumber => setsOfValues[outerNumber][innerNumber])
    )
  )
);

threeStreamsOfThings$.pipe(flatMap(value => value))
```

**Let's try switchMap**

switchMap behaves differently. Modify the last line to look like this:
```js
threeStreamsOfThings$.pipe(switchMap(value => value))
```

When the second stream starts emitting numbers, switchMap only puts the numbers from the second stream into the flattened stream. When the third stream starts emitting emojis, the flattened stream only includes emojis from then on. All other incoming events from the first two streams are discarded. Compare that to flatMap, where all the types of values were mixed together when the streams overlapped.

**Checklist**

Each time you check or uncheck a box, this application fakes an HTTP request to a server to save the change. Since each request results in a new stream for the HTTP request being created inside the stream of click events, we have to flatten the result to apply it to our UI.

```js
const { fromEvent, from } = rxjs;
const { debounceTime, map, tap, switchMap, flatMap } = rxjs.operators;

const fieldset = document.querySelector("fieldset");
const resultsArea = document.querySelector(".results-area");

const eventToRequestBody = event => ({
  status: event.target.checked,
  id: event.target.value,
  name: event.target.name
});

const changes1$ = fromEvent(fieldset, "input");

changes1$
  .pipe(
    map(eventToRequestBody),
    tap(requestBody => setItemInteractivity(requestBody.id, false)),
  
  // what happens if you use switchMap here? Answer is at the bottom of the pen
    flatMap(requestBody => {
      return from(fakeSave(requestBody)).pipe(
        map(saveResponse => ({ ...requestBody, response: saveResponse }))
      );
    }),
   tap(responseInfo => setItemInteractivity(responseInfo.id, true)),
    map(JSON.stringify)
  )
  .subscribe(text => (resultsArea.innerText = text));

const setItemInteractivity = (id, active) => {
  document.getElementById(id).disabled=!active
   
  const spinner = document.querySelector(`label[for=${id}] + div`)
  active ? spinner.classList.add('hidden') : spinner.classList.remove('hidden')
}


/**
 * End of real frontend code.
 * This stuff provides a fake backend - an async function that returns results of writing data
 */
function roughDelay() {
  return new Promise(resolve =>
    setTimeout(resolve, 100 + Math.random() * 3000)
  );
}

function fakeSave(data) {
  return roughDelay().then(() => {
    return Math.random() > 0.7;
  });
}

// if we use switchMap here, the app only cares about the most recent http request, and activity indicators for previous http requests aren't removed when the response completes if multiple updates are happening at once
// switchMap(requestBody => {


// since flatMap does not discard any events, every http request that finishes stays in the stream
// flatMap(requestBody => {
```

The internet can be slow and unreliable, so responses may come back in a different order than their requests were sent, and some of them will be errors. This means flatMap should be used to flatten the stream so we that receive all the responses. What do you think would happen if we chose switchMap? Edit the code and find out.

**Type-ahead Search Field**

After typing a few characters into this search field, it fakes an HTTP request to a server to get a list of cities you may be searching for. Like the above example, that results in a nested stream (HTTP response inside a stream of input change events), so the result needs to be flattened to be used easily in the UI.

```js
const { fromEvent } = rxjs;
const { debounceTime, tap, switchMap, flatMap } = rxjs.operators;

const searchField = document.querySelector("[name=search]");
const resultsArea = document.querySelector(".results-area");

const changes$ = fromEvent(searchField, "input");

changes$
  .pipe(
    // debouncing is a good idea, but makes it harder demonstrate the point of this post
    // debounceTime(400),

    // what would happen if you used flatMap instead of switchMap here? Answer is at then end of the source code
    switchMap(event => fakeSearch(event.target.value))
  )
  .subscribe(result => {
    resultsArea.innerText =
      JSON.stringify(result, null, 2);
  });

/**
 * End of real frontend code.
 * This stuff provides a fake backend - an async function that returns search results
 */

function roughDelay() {
  return new Promise(resolve =>
    setTimeout(resolve, 250 + Math.random() * 3000)
  );
}

function fakeSearch(keyword) {
  return roughDelay().then(() => {
    return fuse.search(keyword);
  });
}

var options = {
  caseSensitive: false,
  shouldSort: true,
  threshold: 0.6,
  location: 0,
  distance: 100,
  maxPatternLength: 32,
  minMatchCharLength: 1,
  keys: ["name"]
};

const list = [
  "Toronto",
  "Montreal",
  "Calgary",
  "Ottawa",
  "Edmonton",
  "Mississauga",
  "Winnipeg",
  "Vancouver",
  "Brampton",
  "Hamilton",
  "Quebec City",
  "Surrey",
  "Laval",
  "Halifax",
  "London",
  "Markham",
  "Vaughan",
  "Gatineau",
  "Saskatoon",
  "Longueuil",
  "Kitchener",
  "Burnaby",
  "Windsor",
  "Regina",
  "Richmond",
  "Richmond Hill",
  "Oakville",
  "Burlington",
  "Greater Sudbury",
  "Sherbrooke",
  "Oshawa",
  "Saguenay",
  "Lévis",
  "Barrie",
  "Abbotsford",
  "Coquitlam",
  "Trois-Rivières",
  "St. Catharines",
  "Guelph",
  "Cambridge",
  "Whitby",
  "Kelowna",
  "Kingston",
  "Ajax",
  "Langley",
  "Saanich",
  "Terrebonne",
  "Milton",
  "St. John's",
  "Thunder Bay",
  "Waterloo",
  "Delta",
  "Chatham-Kent",
  "Red Deer",
  "Strathcona County",
  "Brantford",
  "Saint-Jean-sur-Richelieu",
  "Cape Breton",
  "Lethbridge",
  "Clarington",
  "Pickering",
  "Nanaimo",
  "Kamloops",
  "Niagara Falls",
  "Victoria",
  "Brossard",
  "Repentigny",
  "Newmarket",
  "Chilliwack",
  "Maple Ridge",
  "Peterborough",
  "Kawartha Lakes",
  "Drummondville",
  "Saint-Jérôme",
  "Prince George",
  "Sault Ste. Marie",
  "Moncton",
  "Sarnia",
  "Wood Buffalo",
  "New Westminster",
  "Saint John",
  "Caledon",
  "Granby",
  "St. Albert",
  "Norfolk County",
  "Medicine Hat",
  "Grande Prairie",
  "Airdrie",
  "Halton Hills",
  "Port Coquitlam",
  "Fredericton",
  "Blainville",
  "Saint-Hyacinthe",
  "Aurora",
  "North Vancouver",
  "Welland",
  "North Bay",
  "Belleville",
  "Mirabel"
].map(name => ({ name }));

const fuse = new Fuse(list, options);

// flatMap results in every response from the server being rendered, even outdated responses (since they don't always come back  to the browser in the order they were requested)
// flatMap(event => fakeSearch(event.target.value))

// switchMap ignores results from outdated search requests, which gives us the desired behaviour
// switchMap(event => fakeSearch(event.target.value))

```

You may type faster than the server can respond to search requests, and we only care about the most recent request's response. If an older response from an outdated request arrives with stale data, we can discard it. This is a great place to apply switchMap. What do you think would happen if you chose flatMap in this use case instead? Edit the code and find out.
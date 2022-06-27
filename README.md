# Observables and RxJS Operators

A collection of thoughts with the aim to bring you observables and RxJS operators closer.

Every time I have to explain the most used operators to a colleague,
I was hindered by how the official (https://rxjs.dev/) and nearly official docs fail to communicate real use cases.
This is my attempt to gather everything in a sharable document.

The examples are constructed from my real life experience with RxJS in Angular with TypeScript
but I absolutely do not claim correctness or completion.

## General concept

Consider an observable like a marble run.
Emitted values (aka events) are like marbles going through a series of pipes and boxes.
At the bottom the marble falls out and this is the result you consume at a subscriber.

```mermaid
graph LR;
    Source Observable -- Value --> Subscriber;
```

## Code examples
All code examples are part of an Angular component written in TypeScript.
The base layout looks as follows:

```typescript
Example component
```
```angular2html
<div>?</div>
```
 
The examples are constructed so that the code should be pluggable into the base layout.
Services need to be added to the constructor to be accessible and are meant as realistic observable providers.
The internals of such services is revealed further below.
These should not matter for the understanding of Observables itself.

## Observables

TODO: emitted value, subscriber, only emits when there are subscribers

## Operators

TODO: Keep steps separated to make them more readable
TODO: Subscribing (subscribe vs async pipe)

### tap
__Observe the values without interfering.__

This is the most simple operator as it works like a looking glass.
Access the value to e.g. log it or have a side-effect (assign it to another field). 
```mermaid
graph LR;
    src[Source Observable] -- Value --> op[tap()] --> sink[Subscriber];
```
```typescript
readonly userName$: Observable<string> = this.accountService.getUserName().pipe(
    tap(userName => console.log(username))
);
```
```angular2html
<div>
    <span translate>user.name</span> <span>{{userName$ | async}}</span>
</div>
```

### map
__Convert the emitted value.__

This is a simple transformation of the emitted value to another.
Very useful when you want to e.g. extract a field from an object.

Marble speak: The marble in a lane is replaced by this box - in goes a red marble, out comes a blue marble.

```mermaid
graph LR;
    src[Source Observable] -- Value --> op[map()] -- Another Value --> sink[Subscriber];
```
```typescript
readonly userName$: Observable<string> = this.accountService.getUser().pipe(
    map(user => user.name)
);
```
```angular2html
<div>
    <span translate>user.name</span> <span>{{userName$ | async}}</span>
</div>
```

### switchMap
__Replace the observable.__

With `switchMap` we don't change the value as in `map` but the whole observable.
Instead of having access to the value inside an observable we work on the level of the observable. 

The primary use case is to chain multiple asynchronous calls together.
E.g. we have an observable param$ and with that information we want to make a call to the backend. 

Marble speak: We replace the whole lane instead of only the marble itself.

```mermaid
graph LR;
    src[Source Observable] -- Value --> op[switchMap()] -- Another Value --> sink[Subscriber];
```
```typescript
readonly selectedData$: Observable<Data> = this.params$.pipe(
    switchMap(params => this.dataService.getDataById(params.id))
);
```
```angular2html
?
```

### combineLatest
__Gather multiple observables so that you can use all values at once__

When you need data from different observables to generate a result this is the way.
Only emits when all observables have fired. => You are guaranteed that all values are present.

Marble speak: Multiple lanes are entering this box
which will release a marble when in all lanes a marble reached the box. 

```mermaid
graph LR;
    src[A] -- Value1 --> op[combineLatest()] -- new ValueFrom123 --> sink[Subscriber];
    src[B] -- Value2 --> op[combineLatest()];
    src[C] -- Value3 --> op[combineLatest()];
```
```typescript
readonly selectedData$: Observable<Data> = combineLatest([this.params$, this.dataService.getAllData().pipe(
    map(([params, allData]) => allData.find(dataEntry => dataEntry.id === params.selectedDataId))
);
```
```angular2html
?
```


## Services
TODO: Reveal internals and how services can be implemented with http calls, caching, interfaces, constructors, etc.
TODO: sharereplay
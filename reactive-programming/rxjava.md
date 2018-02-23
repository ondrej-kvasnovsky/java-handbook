# RxJava

[RxJava](https://github.com/ReactiveX/RxJava) is implementation of [ReactiveX](http://reactivex.io). ReactiveX is a library for composing asynchronous and event-based programs by using observable sequences.

Callbacks are too low level and CompletableFuture deals only with single value, there comes streaming event driven and declarative API of RxJava.

### Observable

Observable is generalisation of Observer pattern. Observable adds stream-like API, API with common operations and back pressure or throttling. 

Java streams should be used when we know what data are going to be processed, something like batch processing. While Observable is more for situations when we do not know what is coming, something like on demand computations.

 

### Flowables

Flowable has similar API as Observable anrovides reactive pull back pressure. 


## observable
Every observable is just a sequence.  

An observable has **_3_** types of event:  

	  .Next  
	  .Error
	  .Completed

#### Creating
There are serveral ways to create observable sequences  

*  never  
```Swift  
	// A sequence that never terminate and never emits any event
	let neverSequence = Observable<String>.never()
	// Output:
	// 
```
*  empty
```Swift  
	// A sequence that only emit .completed event
	let neverSequence = Observable<String>.empty()
	// Output:
	// Complete
	//
```
*  just
```Swift  
	// A sequence with a single element
	Observable.just("🔴")
	// Output:
	// Next(🔴)
	// Completed
	//
```
*  of
```Swift  
  // A sequence that with a fixed number of elements
  let neverSequence = Observable<String>.of("🐶", "🐱", "🐭", "🐹")
  // Output:
  // Next(🐶)  
  // Next(🐱)  
  // Next(🐭)  
  // Next(🐹)  
  // Completed  
  //
```
* toObservable
```Swift
  // Create an observable sequence from SequenceType, such as Array, Dictionary or Set
  ["🐶", "🐱", "🐭", "🐹"].toObservable()
  // Output:
  // 🐶
  // 🐱
  // 🐭
  // 🐹
  //
```
* create
```Swift
	// Create a custom observable sequence
	let myJust = { (element: String) -> Observable<String> in
        return Observable.create { observer in
            observer.on(.Next(element))
            observer.on(.Completed)
            return NopDisposable.instance
        }
    }
	// Output:
	// Next(🔴) 	
	// Completed
	//
```
* range
```Swift
	// An observable sequence that emits a range of integers 
	Observable.range(start: 1, count: 2)
	// Output:
	// 1
	// 2
	// Completed
```
* repeat
```Swift
	// An observable sequence that emits the given element indefinitely
	Observable.repeatElement("🔴").take(1)
	// Output:
	// 🔴
```
* generate
```Swift
	// Generate an observable sequence with the given condition
	Observable.generate(
            initialState: 0,
            condition: { $0 < 2 },
            iterate: { $0 + 1 }
        )
 	// Output:
 	// 0
 	// 1
 	//
```
* defered
```Swift
	// An observable sequence for each subscriber
	let deferredSequence = Observable<String>.deferred {
        return Observable.create { observer in
            print("Emitting...")
            observer.onNext("🐶")
            observer.onNext("🐱")
            return NopDisposable.instance
        }
    }
	// Output:
	// Assume there are two subscriber
	// 1
	// Emitting...
	// 🐶
	// 🐱
	// 2
	// Emitting...
	// 🐶
	// 🐱
	//
```
* error
```Swift
	// An observable sequence that emits no event and immediately terminate with error 
	Observable<Int>.error(Error.Test)
	// Output:
	// Error
	//
```
* doOn
```Swift
	Observable.of("🍎", "🍐", "🍊", "🍋").doOn { print("Intercepted:", $0) }
	// Output:
	// Intercepted: Next(🍎)
	// 🍎
	// Intercepted: Next(🍐)
	// 🍐
	// Intercepted: Next(🍊)
	// 🍊
	// Intercepted: Next(🍋)
	// 🍋
	// Intercepted: Completed
	//
```

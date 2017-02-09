### Subject
#### PublishSubject
Emit new events to all subscriber as of their time of the subscribtion
```Swift
  let subject = PublishSubject<String>()
  subject.addObserver("1").addDisposableTo(disposeBag)
  subject.onNext("🐶")
    
  subject.addObserver("2").addDisposableTo(disposeBag)
  subject.onNext("🅰️")
  // Output:
  // 1
  // Next(🐶)
  // 1
  // Next(🅰️)
  // 2
  // Next(🅰️)
  //
```
#### ReplaySubject
Emit new events to all subscriber with the sepecified bufferSize number of previous events.
```Swift
  let replaySubject = ReplaySubject<String>.create(bufferSize: 2)
  replaySubject.subscribe {
    print("1")
    print($0)
  }.addDisposableTo(disposeBag)
    
  replaySubject.onNext("🐶")
  replaySubject.onNext("🐱")
    
  replaySubject.subscribe {
    print("2")
    print($0)
  }.addDisposableTo(disposeBag)
    
  replaySubject.onNext("🅰️")
  // Output:
  // 1
  // Next(🐶)
  // 1
  // Next(🐱)
  // 2
  // Next(🐶)
  // 2
  // Next(🐱)
  // 1
  // Next(🅰️)
  // 2
  // Next(🅰️)
  // 
```
#### BehaviorSubject
Emit new events to the subscriber, including the most recent event
```Swift
  let behaviorSubject = BehaviorSubject(value: "Empty")
  
  behaviorSubject.subscribe {
    print("1")
    print($0)
  }.addDisposableTo(disposeBag)
  
  behaviorSubject.onNext("🐶")
  behaviorSubject.onNext("🐱")
  
  behaviorSubject.subscribe {
    print("2")
    print($0)
  }.addDisposableTo(disposeBag)
  
  behaviorSubject.onNext("🅰️")
  // Outpur:
  // 1
  // Next(Empty)
  // 1
  // Next(🐶)
  // 1
  // Next(🐱)
  // 2
  // Next(🐱)
  // 1
  // Next(🅰️)
  // 2
  // Next(🅰️)
  //
```
#### Variable
Wrap a _BehaviorSubject_, and emit a completed event when it about to be disposed of
```Swift
  let variable = Variable("Empty")
  
  variable.asObservable().subscribe {
    print("1")
    print($0)
  }.addDisposableTo(disposeBag)
  
  variable.value = "🐶"
  variable.value = "🐱"
  
  variable.asObservable().subscribe {
    print("2")
    print($0)
  }.addDisposableTo(disposeBag)
  
  variable.value = "🅰️"
  // Output:
  // 1
  // Next(Empty)
  // 1
  // Next(🐶)
  // 1
  // Next(🐱)
  // 2
  // Next(🐱)
  // 1
  // Next(🅰️)
  // 2
  // Next(🅰️)
  // 1
  // Completed
  // 2
  // Completed
  //
```

`BehaviorSubject` `PublishSubject` and `ReplaySubject` will not automatically emit completed events when they are about to be disposed of

### Combination operator
#### startWith
Emit specified sequence of elements before beginning to emit the elements from the source sequence.  
In other way, it can be chained on a LIFO basis.
```Swift
  Observable.of("🐶", "🐱", "🐭", "🐹")
    .startWith("1️⃣")
    .startWith("2️⃣")
    .startWith("3️⃣", "🅰️", "🅱️")
    .subscribeNext { print($0) }
    .addDisposableTo(disposeBag)
  // Output:
  // 1️⃣ 2️⃣ 3️⃣ 🅰️ 🅱️ 🐶 🐱 🐭 🐹
```
#### zip
Combine the source sequence into a single sequence, and will emit from the combined sequence elements from the each of the source sequence at the corresponding index  
```Swift
  let subject1 = PublishSubject<String>()
  let subject2 = PublishSubject<Int>()
  
  Observable.zip(subject1, subject2) { string, integer in
    "\(string) - \(integer)"
    }.subscribeNext { print($0) }.addDisposableTo(disposeBag)
  
  subject1.onNext("A")
  subject1.onNext("B")
  subject2.onNext(1)
  subject1.onNext("C")
  subject2.onNext(2)
  // Output:
  // A - 1
  // B - 2
  //
```
#### combineLast
![](https://raw.githubusercontent.com/Tangdixi/MyNote/master/RxSwift/combineLast.png)

### Transform Operator

#### map
Applies a transforming closure to elements emitted by an observable sequence, and return a new observable sequence of the transformed elements

```Swift
  Observable.of(1, 2, 3).map { $0 * $0 }
  // Output:
  // 1
  // 4
  // 9
  //
```

#### flatMap
Transform the items emitted by an observable into observables, then flatten the emissions from those into a single observable
![](https://raw.githubusercontent.com/Tangdixi/MyNote/master/RxSwift/flatMap.png)

#### flatMapLatest
FlatMapLatest is equal to map + switchLast

#### scan and reduce
scan  

![](https://raw.githubusercontent.com/Tangdixi/MyNote/master/RxSwift/scan.png)  
  
reduce  

![](https://raw.githubusercontent.com/Tangdixi/MyNote/master/RxSwift/reduce.png)

# 🧮 계산기 [STEP 2]
> 피연산자와 연산자 리스트를 문자열 형태로 전달하면 이를 연산자 기준으로 분리하고 숫자(Double)로 변환하여 연산을 수행하는 로직을 구현합니다.
>> **비고사항**
>> 1. <details><summary>(Toggle) <U>정해진 UML</U>을 기반으로 타입을 생성하였습니다.</summary><div markdown="1"><img src="https://hackmd.io/_uploads/ByY8jQJvn.png"></div></details>
>> 2. STEP 1에서 구현한 Queue를 활용하였습니다.



<br><br>
## 목차📌
1. [팀원소개](##팀원소개👩‍💻)
2. [타임라인](##타임라인🗓️)
3. [다이어그램](##다이어그램📊)
4. [트러블슈팅](##트러블슈팅🚨)
5. [참고자료](##참고자료📘)
6. [회고](##회고📝)

<br><br>
## 팀원소개👩‍💻
| <img src="https://hackmd.io/_uploads/r1rWKewLn.png" width="200"/> |
| :-: |
| [**maxhyunm**](https://github.com/maxhyunm)<br/> |

<br><br>
## 타임라인🗓️
작업 진행 기간 | 23.06.01.(목) ~ 23.06.08.(금)

| 날짜 | 진행 사항 |
| -------- | -------- |
| 23.06.01.(목)     | Operator 타입 생성<br/>↳ add(), subtract(), multiply(), divide() 메서드 테스트 / 구현 / 리팩토링 <br/><br/>Formula 타입 생성<br/>↳ result() 메서드 테스트 / 구현 / 리팩토링<br/><br/>EspressionParsor 타입 생성<br/>↳ parse() 메서드 테스트 / 구현 / 리팩토링<br/>↳ componentsByOperators() 메서드 테스트 / 구현 / 리팩토링|
| 23.06.02.(금)     |ExpressionParser 리팩토링, 테스트 추가<br/>CalculatorItemNode에 제네릭 설정 추가<br/>파일 위치 정리      |
| 23.06.03.(토)     |제네릭 타입명 수정(T->Element)     |
| 23.06.05.(월)     |필요없는 변수 제거, 개행 추가     |
| 23.06.08.(목)     |README 작성      |


<br><br>
## 트러블슈팅🚨

### 1️⃣ 타입캐스팅 <br>
🔒 **문제점**
>연산을 위한 Queue는 CalculateItem 형태로 요소를 저장하고, 실제 연산은 CalculateItem을 준수하는 Double과 Operator 타입을 기반으로 이루어지다 보니 매번 다운캐스팅을 해야 하는 번거로움이 있었습니다.
```swift=
while true {
    do {
        guard let rhs = try operands.dequeue() as? Double else {
            throw CalculatorError.invalidInput
        }
        ...
        guard let newOperator = try operators.dequeue() as? Operator else {
            throw CalculatorError.invalidOperator
        }
    ...

```


🔑 **해결방법**
>QueueType에 associatedtype을 추가하고, 해당 프로토콜을 준수하는 타입들과 CalculateItemNode 타입에 제네릭을 추가하여 타입캐스팅 문제를 해결하였습니다.
```swift=

protocol QueueType {
    associatedtype Item where Item: CalculateItem
    
    mutating func enqueue(_ value: Item)
    mutating func dequeue() throws -> Item
    mutating func removeAll()
}
```
```swift=
class CalculatorItemNode<Element: CalculateItem> {
    private(set) var value: Element
    private(set) var next: CalculatorItemNode?
    
    init(_ value: Element) {
        self.value = value
    }
    ...
```
---
### 2️⃣ parse() 마이너스 부호 처리<br>
🔒 **문제점**<br>
>빼기 연산자와 마이너스부호를 구분하는 방법에 있어 고민이 많았습니다. 처음에는 마이너스 부호를 "!"로 변환하여 진행 후 다시 변환하는 방식으로 구현했지만, 너무 비효율적이었습니다.
```swift=
static func parse(from input: String) throws -> Formula {
    ...
    let operatorWithMinus = [(before: "+-", after: "+!"),
                             (before: "--", after: "-!"),
                             (before: "/-", after: "/!"),
                             (before: "*-", after: "*!")]
    let inputValue = operatorWithMinus.reduce(input) { inputString, operatorType in
        inputString.replacingOccurrences(of: operatorType.before, with: operatorType.after)
    }
    ...
}

static private func componentsByOperators(from input: String) -> [String] {
    ...
    inputValues = inputValues.replacingOccurrences(of: "!", with: "-")
    ...
}
```
🔑 **해결방법** <br>
> 스토리보드 버튼 타이틀값으로 들어간 빼기 연산자가 마이너스 부호와 비슷한 형태이지만 살짝 다른 문자열로 되어 있다는 사실을 알았고, 이 문자열을 그대로 활용하면 자연스럽게 마이너스 부호와 구분이 될 것이라는 생각이 들었습니다. 덕분에 복잡했던 로직이 매우 간결해졌습니다.

```swift=
static func parse(from input: String) -> Formula {
    ...
    let operandComponents = self.componentsByOperators(from: input).compactMap { Double($0) }
    let operatorComponents = input.compactMap { Operator(rawValue: $0) }
    ...
}
```
---
### 3️⃣ ExpressionParser 메서드 기능 분리<br>
🔒 **문제점**<br>
>이미 작성된 UML을 토대로 진행하다 보니 각 메서드의 역할이 어떤 것인지 확실히 파악하기가 어려웠습니다. 또한 1번의 문제를 해결하기 전에는 마이너스 부호 부분까지 포함하여 로직이 더욱 복잡하고, 기능별로 메서드가 제대로 구별된 느낌이 들지 않았습니다. 특히 componentsByOperators() 같은 경우 연산자를 토대로 문자열을 쪼개는 기능을 하는 메서드가 맞는지 명확하지 않아 보였습니다.
```swift
static private func componentsByOperators(from input: String) -> [String] {
     let allOperators = Operator.allCases.map { $0.rawValue }

     var inputValues = allOperators.reduce(input) { inputString, operatorType in
         inputString.split(with: operatorType).joined(separator: "|")
     }
     inputValues = inputValues.replacingOccurrences(of: "!", with: "-")
     let result = inputValues.split(with: "|")
     return result
 }
```

🔑 **해결방법** <br>
>메서드의 이름을 토대로 정확히 어떤 역할을 해야하는지를 재정리했고, 이를 기반으로 불필요한 로직을 쳐내어 정리하였습니다. 이 과정을 거치면서 전체적인 로직이 어떤 방향으로 이루어져야 하는지도 조금 더 명확해졌습니다. 또한 return에서 바로 함수 결과를 리턴할 수 있도록 하여 불필요한 변수 사용을 줄였습니다.
```swift=
static private func componentsByOperators(from input: String) -> [String] {
    return Operator.allCases.reduce([input]) { resultArray, operatorItem in
        resultArray.map { $0.split(with:operatorItem.rawValue) }.flatMap { $0 }
    }
}

extension String {
    func split(with target: Character) -> [String] {
        return components(separatedBy: String(target))
    }
}
```
---
### 🤔 고민했던 점 <br>
> 이미 정리된 UML을 토대로 타입과 메서드의 역할을 추론하며 작업하는 방식이 익숙하지 않아서 평소보다 대대적인 범위로 리팩토링을 반복했던 스텝입니다. 처음에는 TDD를 준수하려 했지만, 이 과정에서 자연스럽게 선 구현 후 유닛테스트 형식으로 진행하게 되었습니다. 이 부분을 개선하기 위해서는 작업에 들어가기 전에 각 타입에 대한 충분한 고민과 상세한 정리가 필요할 것 같다는 생각이 들었습니다.


<br><BR>
## 참고자료📘
- [The Swift Programming Language - Generics](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/)
- [Apple Developer Documentation - reduce(_: _:)](https://developer.apple.com/documentation/swift/array/reduce(_:_:))
- [Apple Developer Documentation - map(_:)](https://developer.apple.com/documentation/swift/array/map(_:)-87c4d)
- [Apple Developer Documentation - flatMap(_:)](https://developer.apple.com/documentation/swift/array/flatmap(_:)-6chu8)

<br><BR>
## 회고📝
👍 익숙하지 않았던 제네릭 사용과 고차함수 사용을 통해 다양한 구현에 성공한 점<br/>
👍 시간 내에 잘 마무리한 점<br/>
👎 사전에 타입과 메서드에 대한 파악을 충분히 진행하지 않아 많은 수정을 거친 점<br/>




# 🧮 계산기 [STEP 1]
> CalculateItem 타입의 데이터를 받아 선입선출 방식으로 쌓는 Queue를 구현합니다.
> Queue에는 입력(enqueue)과 출력(dequeue), 전체 삭제(removeAll) 기능이 있습니다.
> 전체 작업 내용은 TDD를 기반으로 진행합니다.

<br><br>
## 목차📌
1. [팀원소개](##팀원소개👩‍💻)
2. [타임라인](##타임라인🗓️)
3. [다이어그램](##다이어그램📊)
4. [트러블슈팅](##트러블슈팅🚨)
5. [참고자료](##참고자료📘)

<br><br>
## 팀원소개👩‍💻
| <img src="https://hackmd.io/_uploads/r1rWKewLn.png" width="200"/> |
| :-: |
| [**maxhyunm**](https://github.com/maxhyunm)<br/> |

<br><br>
## 타임라인🗓️
작업 진행 기간 | 23.05.29.(월) ~ 23.06.01.(목)

| 날짜 | 진행 사항 |
| -------- | -------- |
| 23.05.29.(월)     | UML 작성<br/><br/>CalculatorItemQueue 구조체 생성<br/>↳ enqueue() 메서드 테스트 / 구현 / 리팩토링<br/>↳ dequeue() 메서드 테스트 / 구현 / 리팩토링<br/>↳ removeAll() 메서드 테스트 / 구현 / 리팩토링<br/><br/>CalculatorItemNode 클래스 생성<br/>↳ addNext() 메서드 테스트 / 구현 / 리팩토링<br/><br/>CalculateItem 프로토콜 생성<br/>↳ String, Int, Double 익스텐션 추가<br/><br/>CalculatorErroe 열거형 생성|
| 23.05.30.(화)     |전체 메소드 테스트 및 리팩토링      |
| 23.05.31.(수)     |CalculatorItemQueue 테스트 더블 생성<br/>↳ 원본 구조체의 접근제어 private으로 지정     |
| 23.06.01.(목)     |README 작성      |

<br><br>
## 다이어그램📊
![](https://hackmd.io/_uploads/ByPsRzLL3.png)


<br><br>
## 트러블슈팅🚨

### 1️⃣ Queue 구현 <br>
🔒 **문제점**<br>
>처음에는 양방향 링크드리스트 방식으로 CalculatorItemQueue를 구현하기 위해 각 노드가 바로 전 노드와 다음 노드의 주소를 previous/next로 갖고 있도록 만들었습니다. 하지만 계산기가 작동하면서 선입선출 이외의 방식으로 내부 요소를 검색하거나 추출할 일이 없다는 사실을 깨달았습니다.

```swift!
// CalculatorItemNode.swift
    
class CalculatorItemNode {
    var value: CalculateItem
    var next: CalculatorItemNode?
    var previous: CalculatorItemNode?
    
    init(_ value: CalculateItem) {
        self.value = value
    }
    
    func changeNext(_ next: CalculatorItemNode) {
        self.next = next
    }
    
    func changePrevious(_ previous: CalculatorItemNode) {
        self.previous = previous
    }
}
```

🔑 **해결방법** <br>
>메모리 사용이 덜하고 순환참조의 위험이 낮은 단방향 링크드리스트로 변경하여 작업하였습니다.
```swift=
// CalculatorItemNode.swift

class CalculatorItemNode {
    private(set) var value: CalculateItem
    private(set) var next: CalculatorItemNode?
    
    init(_ value: CalculateItem) {
        self.value = value
    }
    
    func changeNext(_ next: CalculatorItemNode?) {
        self.next = next
    }
}
```
---
### 2️⃣ 접근제어 설정 <br>
🔒 **문제점**<br>
>CalculatorItemQueue의 프로퍼티들은 private(set)으로 읽기 전용 설정을 해 둔 상태였습니다. 하지만 테스트를 제외하면 외부에서 해당 프로퍼티에 접근할 일이 없었기 때문에 고민이 되었습니다.
```swift=
// CalculatorItemQueue.swift

struct CalculatorItemQueue {
    private(set) var head: CalculatorItemNode? = nil
    private(set) var tail: CalculatorItemNode? = nil
    private(set) var count: Int = 0
    
    ...
    
```
🔑 **해결방법** <br>
>Queue 타입을 프로토콜로 분리하여 CalculatorItemQueueFake라는 테스트 더블 객체를 생성하였고, 원본의 접근제어는 private으로 설정하여 문제를 해결하였습니다.
```swift=
// CalculatorItemQueue.swift

protocol QueueType {
    mutating func enqueue(_ value: CalculateItem)
    mutating func dequeue() throws -> CalculateItem
    mutating func removeAll()
}

struct CalculatorItemQueue: QueueType {
    private var head: CalculatorItemNode? = nil
    private var tail: CalculatorItemNode? = nil
    private var count: Int = 0

    ...
    

// CalculatorItemQueueFake.swift
    
struct CalculatorItemQueueFake: QueueType {
    var head: CalculatorItemNode? = nil
    var tail: CalculatorItemNode? = nil
    var count: Int = 0
    
    ...
    
```
---
### 3️⃣ CalculateItem 프로토콜 구현 <br>
🔒 **문제점**
>CalculatorItemQueue의 요소들이 CalculateItem으로 이루어져야 한다는 제약조건이 있어, 테스트를 진행하기가 쉽지 않았습니다. 또한 추후 숫자와 연산자를 받아 계산하는 기능을 추가해야 하기 때문에, 이 부분에 대한 고민이 필요했습니다.


🔑 **해결방법**
>extension을 통해 String과 Int, Double 등 계산에 활용될 만한 타입들을 확장하여 CalculateItem을 준수하도록 선언하였습니다.
```swift=
// CalculateItem.swift

protocol CalculateItem {}

extension Int: CalculateItem {}
extension String: CalculateItem {}
```
---
### 🤔 고민했던 점 <br>
>TDD의 흐름에 맞춰 작업을 하려다 보니 구현 후 리팩토링을 하면서 많은 내용이 바뀌어 해당 테스트를 재작성해야 하는 경우가 빈번히 발생하였습니다. 때문에 시간이 많이 소요되었는데, 이 부분은 사전에 UML을 통해 최대한 꼼꼼히 설계를 마친 후 작업에 들어가면 좋았을 것 같다는 생각이 들었습니다.


<br><BR>
## 참고자료📘
- [Protocol](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/)
- [Extensions](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions/)
- [XCTest](https://developer.apple.com/documentation/xctest/)
- [UML](https://sparxsystems.com/enterprise_architect_user_guide/15.2/model_domains/whatisuml.html)
- [ARC](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)






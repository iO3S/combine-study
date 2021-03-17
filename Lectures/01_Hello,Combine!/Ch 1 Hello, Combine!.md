# Ch.1: Hello, Combine!

## Table of Contents

 ㄱ. [비동기 프로그래밍]()

 ㄴ. [Foundation, UIKit/AppKit]()

 ㄷ. [Combine의 근간]()

 ㄹ. [Combine 기초]()

  1. [Publishers]()
  2. [Operators]()
  3. [Subscribers]()
  4. [Subscriptions]()

 ㅁ. ["일반(standard)" 코드 대비 Combine 코드의 장점은?]()

 ㅂ. [앱 구조]()

 ㅅ. [Book Prejects]()

 ㅇ. [Key Points]()

 

*이 책은 Combine 프레임워크와 Apple 플랫폼에서 Swift를 통해 선언적이고 반응형 앱을 작성하는 방법을 소개합니다.*

Apple은 Combine에 대해서 다음과 같이 설명합니다.

> Combine 프레임워크는 앱이 이벤트를 처리하는 방식에 대해서 선언적 접근 방식을 제공한다. 여러개의 Delegate Callback 또는 Completion Handler Closure를 구현하는 대신 특정 이벤트 소스에 대해 단일 Processing Chain을 작성할 수 있다. Chain의 각 부분은 이전 단계에서 수신한 요소에 대해 별개의 작업을 수행하는 결합 연산자들을 의미한다.

비록 명확하고 요점이 정리된 설명이지만, 처음에는 추상적으로 들릴 수 있습니다. 
그렇기 때문에 다음 챕터에서 프로젝트를 진행하기 전에 Combine에서 사용하는 도구에들에 대해 간단히 알아봅시다.

# ㄱ. 비동기 프로그래밍

간단하게 단일 쓰레드 언어에서는 한 줄씩 순차적으로 코드가 실행됩니다. 

- **Example**

    ```swift
    begin
    	var name = "Kyung-Jun"
    	print(name)
    	name += " Min"
    	print(name)
    end
    ```

동기식 코드는 이해하기 쉽습니다. 특히 Data의 상태를 설명하기에 쉽죠. 단일 쓰레드로 실행하면 항상 현재의 데이터 상태를 확인할 수 있습니다. 위 코드는 항상 "Kyung-Jun" 을 print 하고 " Min" 을 print 하겠죠.

그렇다면 멀티 쓰레드 언어를 작성한다고 생각해봅시다. UIKit과 Swift로 작성된 iOS 앱과 같이 비동기 이벤트 중심 UI framework를 실행하는 것처럼요. 다음과 같은 코드를 살펴보죠.

- **Example**

    ```swift
    --- Thread 1 ---
    begin
      var name = "Kyung-Jun"
      print(name)

    --- Thread 2 ---
    name = "Haribo"

    --- Thread 1 ---
      name += " Min"
      print(name)
    end
    ```

이렇게 `name` 에 `"Kyung-Jun"`이라는 값을 주고 그 다음에 `"Harding"` 을 더해줬습니다. 이전과 똑같이요. 하지만 동시에 동작하는 다른 쓰레드가 있기때문에 `name` 값이 `"Haribo"` 로 설정될 수도 있는거죠. 코드가 다른 코어에서 동시에 실행될 때는 코드의 어떤 부분이 먼저 수정되거나 실행될지 알 수 없습니다. 위 예제의 Thread 2 는 다음과 같이 실행됩니다.

- 원래 코드와 다른 CPU 코어에서 정확히 동시에 실행
- `name += "Min"` 직전에 실행하므로 원래 값 `"Kyung-Jun"` 대신 `"Haribo"` 이 표시

이 코드를 실행할 때 발생하는 것은 시스템 로드 상태에 따라 다르며 프로그램을 실행할 때마다 다른 결과가 나타날 수 있습니다. 비동기적으로 동시에 실행되는 코드를 실행하는 순간 변경 가능한 상태 관리는 필수적입니다.

# ㄴ. Foundation, UIKit/AppKit

Apple은 매해 거듭하여 비동기 프로그래밍 방식을 개선해왔습니다. 모바일 앱을 작성하는 데 매우 기본적인 사항이기 때문에 우리 대부분은 이미 그 코드들을 사용하고 있을겁니다. 

그리고 주로 사용하는 건 다음과 같을겁니다.

- **NotificationCenter**: 사용자가 기기 방향을 변경하거나, 소프트웨어 키보드가 화면에 표시/숨김 처리될 때와 같이 특정 이벤트가 발생할 때마다 코드를 실행합니다.
- **The Delegate Pattern**: 다른 object를 대신하여 또는 함께 작동하는 object를 정의할 수 있습니다. 예를 들어 AppDelegate에 새로운 remote notification이 도착하면 어떻게 작동할 것인지 정의할 수 있지만 이 코드가 언제 실행될지, 몇 번 실행될지는 전혀 알 수 없습니다.
- **Grand Central Dispatch** and **Operations**: 수행 할 작업을 추상화 하는데 도움이 됩니다. 이를 사용하여 코드가 순차적으로 실행되도록 직렬 대기열에 예약하거나, 우선 순위가 다른 여러 대기열에서 동시에 여러 작업을 실행할 수 있습니다.
- **Closures**: 전달할 수 있는 코드 블럭을 만들어 다른 object가 코드 블럭의 실행 여부와 횟수 및 위치를 결정할 수 있도록 합니다.

일반적으로 UI 이벤트 또는 일부 코드는 비동기식으로 수행하기 때문에 앱 코드 전체가 어떤 순서로 실행될지 가정하는 것은 사실상 불가능합니다. 또한 괜찮은 비동기식 프로그램 작성은 쉽지 않습니다. 비동기 코드와 리소스 공유는 재현이나 추적이 어렵고 궁극적으로 수정하기도 어려운 문제를 일으킬 수 있기 때문이죠. 이러한 문제의 원인은 실제 앱이 각각의 고유한 인터페이스를 가지는 비동기 API들을 사용하는데 있습니다.

![Ch%201%20Hello,%20Combine!/img7.png](Ch%201%20Hello,%20Combine!/img7.png)

Combine은 이렇게 혼란스러운 비동기 프로그래밍 세계를 질서정연하게 정리하는데 도움이 되는 새로운 언어를 Swift 생태계에 도입하는 녀석입니다. Apple은 Combine의 API를 **Foundation framework** 깊숙히 통합시켰기 때문에 `Timer`, `NotificationCenter`, `Core Framework`와 같은 **Core Data** 들은 이미 Combine을 사용하고 있습니다. 

따라서 Combine을 기존의 코드에 도입하는 것은 아주 쉬울겁니다. 또한 Apple은 놀랍고 새로운 UI Framework인 **SwiftUI**를 설계하여 Combine과 쉽게 통합되도록 했습니다. 여기 Apple이 Combine을 사용하여 반응형 프로그래밍을 사용하는 방법을 알 수 있도록 시스템 계층 구조에서 Combine이 어디에 있는지 보여주는 다이어그램이 있습니다.

![Ch%201%20Hello,%20Combine!/img8.png](Ch%201%20Hello,%20Combine!/img8.png)

Foundation에서 SwiftUI에 이르기까지 다양한 시스템 Framework는 Combine에 의존하며 "전통적인" API의 대안으로 Combine에 통합하는것을 제공합니다. 

Combine은 Apple의 framework이므로 `Timer` 또는 `NotificationCenter` 와 같이 이미 테스트로 검증되고 잘 만들어져있는 API를 제거하고 대체하는 것을 목표로 하지 않습니다. Combine 으로 통합되어 서로 비동기식으로 커뮤니케이션하려는 앱의 모든 유형이 Combine을 사용할 수 있도록 합니다.

# ㄷ. Combine의 근간

선언형, 반응형 프로그래밍은 새로운 개념이 아닙니다. 꽤 오래전부터 있던 개념이지만 지난 10년 동안 급부상한 개념입니다. 2009년 Microsoft 팀이 .NET 전용 Reactive Extensions(Rx.NET) 라는 라이브러리를 시작하였고, 이를 2012년에 오픈 소스로 만들었습니다. 그 이후로 많은 언어가 그 개념을 사용하기 시작했고, 현재 RxJS, RxKotlin, RxScala, RxPHP 등 과 같은 많은 Rx 표준이 있습니다. 

Apple의 플랫폼에는 Rx표준을 구현하는 RxSwift와 같은 ThirdParty Framework가 있습니다. Combine은 Rx와 다르지만 Reactive Stream과 유사한 표준을 구현합니다. Reactive Stream은 Rx와 몇 가지 주요한 차이 점이 있지만 둘 다 대부분의 핵심 개념이 유사합니다. 지금까지 위에서 언급한 Rx Framework를 사용하지 않았더라도 걱정할 필요 없습니다. 

지금까지 반응형 프로그래밍은 Apple 플랫폼, 특히 Swift에서 주요 개념이 아니었습니다. 그러나 Apple은 iOS 13 / macOS Catalina 부터 내장 시스템 framework인 Combine을 통해 반응형 프로그래밍 지원을 Apple 생태계에 추가하게 되었습니다.  iOS 13 / macOS Catalina 이상을 지원하는 앱에서만 Combine을 사용할 수 있는 제약은 있지만, 이는 Apple의 다른 기술들처럼 빠르게 지원이 확산되고 최종적으로는 Combine 기술에 대한 수요가 급증하게 될 것 입니다.

# ㄹ. Combine 기초

Combine의 세 가지 핵심 요소는 Publisher, Subscriber, Operator 입니다. Ch.2 Publisher & Subscriber 에서 Subscriber에 대해 자세히 배우고 Section 2 에서는 Operator들을 다룰 것입니다. 여기서는 각각의 타입에 대한 목적과 역할 등 일반적인 내용을 훑어보도록 하겠습니다.

## 1. Publishers

- Publishers는 Subscribers와 같이 하나 이상의 대상에게 시간이 지남에 따라 값을 방출할 수 있는 유형입니다.
- 수학 계산, 네트워킹, 사용자 이벤트 처리 등 모든 Publishers는 여러 이벤트들을 다음 세 가지 유형으로 생성할 수 있습니다.
    1. Publisher의 Generic `Output` 
    2. 성공적인 완료 
    3. Error와 함께 완료된 Publisher의 `Failure` 
- Publisher는 0개 이상의 ouput 값을 가질 수 있으며, 성공 또는 실패로 인해 완료되면 더 이상 이벤트를 생성하지 않습니다.

다음은 `Int` 값을 방출하는 Publisher를 타임 라인에 시각화한 것입니다.

![Ch%201%20Hello,%20Combine!/img9.png](Ch%201%20Hello,%20Combine!/img9.png)

- 위 그림에서 파란색 상자는 타임 라인에서 특정 시간에 방출되는 값을 나타내고 숫자는 방출된 값을 나타냅니다. 그림 오른쪽에 보이는 것과 같은 수직선(|)은 성공적으로 스트림이 완료되었다는 것을 의미합니다.
- 위 Publisher가 다룰 수 있는 이벤트 유형은 매우 보편적이기 때문에 모든 종류의 동적 데이터를 나타낼 수 있습니다. 즉, Combine의 Publisher를 사용하여 앱의 모든 작업을 처리할 수 있습니다.
- Delegate를 추가하거나 Completion Handler를 삽입하는 대신 Publisher를 사용할 수 있습니다.
- Publisher의 가장 좋은 기능 중 하나는 Error 처리 기능이 내장되어 있다는 것입니다. Error 처리는 원하는 경우 마지막에 선택적으로 추가하는 것이 아닙니다.
- 위 그림을 통해 알 수 있듯이 Publisher Protocol은 두 가지 유형을 가집니다.
    - `Publisher.Output`은 Publisher의 방출 값입니다. 만약 Publisher가 `Int`에 특화되었다면 `String`이나 `Date`타입을 방출할 수는 없습니다.
    - `Publisher.Failure`는 Publisher가 뱉을 수 있는 `Error` 타입입니다. 만약 Publisher가 절대 실패하지 않는다면, 이를 `Never`라는 실패 타입으로 표현할 수 있습니다.
- 따라서 주어진 Publisher를 구독할 때 어떤 값이 떨어질지, 실패한다면 어떤 에러가 떨어질지 예상 가능합니다.

## 2. Operators

- Operators는 Publisher Protocol로 선언된 method이며, 선언된 Publisher와 동일하거나 새로운 Publisher로 반환합니다.
- 여러 Operator를 차례로 호출해서 체인을 연결할 수 있기 때문에 매우 유용합니다. 이들은 매우 독립적으로 구성가능하기 때문에 하나의 구독 사이에서 아주 복잡한 로직을 구현하도록 서로 결합될 수 있습니다.
- Operator가 퍼즐 조각처럼 서로 잘 맞도록 하는 방법은 간단합니다. Output이 다음 Input 유형과 일치하지 않으면 결합할 수 없습니다.

![Ch%201%20Hello,%20Combine!/img10.png](Ch%201%20Hello,%20Combine!/img10.png)

- 올바른 Input/Output 유형과 내장된 Error Handling으로 비동기식으로 추상화된 각 작업의 순서를 정의할 수 있다는것은 정말 좋은 부분이다.
- Operator는 항상 Input/Output을 가지고 있으며 일반적으로는 이를 upstream, downstream이라고 합니다. 이를 통해 공유 상태
- Operator는 이전 Operator로 부터 받은 데이터 작업에 중점을 두고 그 결과를 체인의 다음 Operator에게 제공합니다. 즉, 비동기적으로 실행되는 다른 코드는 작업 중인 데이터를 "건너뛰고" 변경할 수 없습니다.

## 3. Subscribers

드디어, Subscription 체인의 끝에 도달했습니다. 모든 Subscription은 Subscribers로 끝납니다. Subscribers는 일반적으로 방출되는 값 또는 완료 이벤트를 통해 "무언가"를 하는 역할입니다. 

![Ch%201%20Hello,%20Combine!/img11.png](Ch%201%20Hello,%20Combine!/img11.png)

- 현재, Combine은 2개의 데이터 스트림 작업을 간단하게 해줄 수 있는 두 개의 빌트인 Subscriber를 제공합니다.
    - **Sink Subscriber**: 출력 값과 완료 이벤트를 수신하는 Closure를 제공합니다. 이것을 통해 원하는 작업들을 수행시킬 수 있습니다.
    - **Assign Subscriber**: 별도로 작성한 코드 없이도 결과 출력물을 데이터 모델 또는 UI 컨트롤의 일부 속성에 바인딩하여 직접 화면에 데이터를 표시 할 수 있게 합니다.
- 데이터에 별도의 요구 사항이 있는 경우 Subscriber를 커스터마이징 하는 것이 Publisher를 커스터마이징 하는 것보다 훨씬 쉽습니다. Combine은 workshop에서 작업에 적합한 툴을 제공하지 않을 때 Custom Tool을 구축할 수 있도록 해주는 매우 간단한 Protocol을 제공합니다.

## 4. Subscriptions

> Note: 여기서는 subscription 이라는 용어를 Combine의 `Subscription Protocol` 뿐만 아니라 해당 `object`, `publisher`, `operator`, `subscriber` 전체 체인을 의미하는 용도로도 사용합니다.

- Subscription의 마지막에 Subscriber를 추가하면 Publisher가 체인의 시작부분 부터 "활성화" 시킵니다. publisher는 Output을 수신할 Subscriber가 없을 경우에는 어떠한 값도 방출하지 않습니다.
- Subscription은 사용자 정의 코드 및 에러처리를 사용하여 비동기 이벤트 체인을 한 번만 선언할 수 있다는 점에서 아주 좋은 개념입니다.
- 즉, Combine을 사용하여 코드를 작성 할 경우, Subscriptions를 통해 앱 전체 로직을 구현하고, 완성하기만 하면 데이터를 push / pull 하거나 다른 Object를 호출하지 않고도 모든 것이 실행 되도록 할 수 있습니다.

![Ch%201%20Hello,%20Combine!/img12.png](Ch%201%20Hello,%20Combine!/img12.png)

- 한번 `Subscription` 코드가 컴파일에 성공하고 작성한 코드에 로직 이슈가 없다면 -끝! 설계된대로 `Subscription`은 사용자의 제스쳐, 타이머가 울리거나 `Publisher`들 중 하나를 활성화 시킬 때마다 비동기식으로 "실행" 될겁니다.
- 또한 `Cancellable`이라는 Combine에서 제공하는 protocol 덕분에 `Subscription`을 메모리로 관리할 필요가 없습니다.
- 시스템에서 제공하는 두 `Subscriber` 모두 `Cancellable Protocol` 을 준수합니다. 즉, `Subscription` 코드 (예: 전체 publisher, operator, subscriber 호출 체인) 가 `Cancellable Object`를 반환합니다. 해당 object를 메모리에서 해제할 때마다 전체 `Subscription`이 취소되고 해당 리소스가 메모리에서 해제됩니다.
*(RxSwift의 Disposable Protocol과 같은 기능?)*
- 예를 들어, ViewController의 Property에 저장하여 Subscription의 수명을 쉽게 "바인딩" 할 수 있습니다. 이렇게 하면 사용자가 View Stack에서 ViewController를 해제하면 해당 Property의 초기화가 해제되고 Subscription도 해제됩니다. 
*(RxSwift의 disposeBag과 같은 기능?)*
- 또는 이 과정을 자동화하기 위해 `Set<AnyCancellable>` 프로퍼티를 설정하고 원하는 만큼 `Subscription`을 집어 넣을 수 있습니다. 이 프로퍼티가 메모리에서 해제되면 그 안의 `Subscription`들도 모두 자동으로 취소 및 해제됩니다.

# ㅁ. "일반(standard)" 코드 대비 Combine 코드의 장점은?

너무 당연한 말이지만 Combine 없이도 최고의 앱을 만들 수 있습니다. 다만 Combine을 사용하는 것은 `Core Data`, `URLSession`, `UIKit`과 같은 추상화를 직접 작성하는 것보다 편리하고 안전하며 효율적입니다.

Combine(및 다른 시스템 프레임워크)은 비동기 코드에 다른 추상화를 추가하는것을 목표로 합니다. 시스템 레벨의 또 다른 추상화 수준은 테스트가 잘되어있으며 지속적이고 안전한 기술을 의미합니다. 

Combine이 프로젝트에 적합한지 여부를 판단하고 결정하는 것은 개발자 각자의 몫입니다. 
하지만 Combine을 채택하면 다음과 같은 혜택이 있습니다.

- Combine은 시스템 레벨에서 통합되었습니다. 즉, Combine 자체는 공개적으로 사용할 수 없는 언어 기능을 사용하기 때문에 사용자가 직접 조작할 수 없는 API형태로 제공 됩니다.
- `Delegate`, `IBAction` 또는 `Closures`와 같은 "오래된" 스타일의 비동기식 코드를 사용하면 버튼이나 제스처의 각 케이스에 대해 사용자가 직접 코드를 작성하도록 하고 이는 테스트할 양이 많아진다는 의미가 됩니다.  Combine은 모든 비동기 연산을 추상화하여 이미 테스트로 검증된 "Operator"를 제공합니다.
- 모든 비동기 작업이 동일한 인터페이스인 `Publisher`를 사용하여 이루어진다면 구성과 재사용성이 아주 강력해집니다.
- Combine의 Operator들은 쉽게 구성될 수 있습니다. 새로운 Operator를 만든다면 나머지 Combine 코드와 즉시 결합되고 사용될 수 있을 것입니다.
- 비동기 코드 테스트는 일반적으로 동기 코드 테스트 보다 복잡합니다. 그러나 Combine을 사용하면 비동기 연산자가 이미 테스트 되어 있기 때문에 비즈니스 로직만 테스트 하면 됩니다. 즉, Subscription이 예상 결과를 출력하는지 여부를 입력하고 테스트하면 됩니다.

이와 같이 대부분의 장점은 안정성과 편의성에 있습니다. Framework가 Apple에서 제공된다는 사실을 생각한다면 Combine 코드 작성에 시간 투자할만하겠죠?

# ㅂ. 앱 구조

- Combine은 앱 구조에 영향을 주는 framework가 아닙니다. MVC, MVVM, VIPER 등 어디서든 사용할 수 있습니다.
- 코드에서 개선하려는 부분에서만 Combine 코드를 선택적으로 추가할 수 있으며 도 아니면 모 식의 선택을 하지 않아도 괜찮습니다. 데이터 모델을 변환하거나 네트워킹 계층을 조정하거나 기존 기능을 그대로 유지하면서 앱에 추가한 새 코드에서만 Combine을 사용하여 시작할 수 있습니다.
- Combine과 SwiftUI를 동시에 채택하면 약간 다른 이야기가 될 수 있습니다. 이 경우 MVC 아키텍처에서 C를 삭제하는 것이 좋습니다만 이는 Combine과 SwiftUI를 함께 사용하기 때문에 요구되는 것입니다. 이 둘은 같은 공간에 있을 때 더욱 간단하게 작동합니다.
- Combine과 SwiftUI를 사용한다면 ViewController는  필요없습니다. 데이터 모델에서 View에 이르기까지 반응형 프로그래밍을 사용하는 경우 View를 제어하기 위해 특별한 Controller가 필요하지 않습니다.

![Ch%201%20Hello,%20Combine!/img13.png](Ch%201%20Hello,%20Combine!/img13.png)

- 더 자세한 내용은 Ch.15 In Practice: SwiftUI & Combine에서 다루겠습니다.

# ㅅ. Book Prejects

- 이 책에서는 먼저 개념부터 시작하여 여러 Operator를 학습하고 시험해보는 단계로 넘어갑니다.
- 다른 System Framework와 달리, Xcode playground에서 Combine을 학습하며 Chapter를 진행하면 더 쉽고 신속하게 테스트 할 수 있으며, 즉시 결과를 확인 할 수 있습니다.

![Ch%201%20Hello,%20Combine!/img14.png](Ch%201%20Hello,%20Combine!/img14.png)

# ㅇ. Key Points

- Combine은 비동기 이벤트를 처리하기 위한 선언적이고 반응적인 프레임워크입니다.
- 비동기식 프로그래밍 통합 도구, 변형 가능한 상태 처리, 오류 처리와 같은 기존 문제를 해결하는 것을 목표로 합니다.
- Combine은 세 가지의 주요 유형을 중심으로 이루어진다.
    - **Publishers**: 시간에 따라 이벤트를 보내는 게시자
    - **Operators**: `upstream`과 `downstream`을 비동기적으로 처리하고 조작하는 운영자
    - **Subscribers**: 결과를 가져와 유용한 작업을 수행하는 소비자

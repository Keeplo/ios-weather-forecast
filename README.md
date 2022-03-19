# 날씨 정보

프로젝트 기간 (2021.9.27 ~ 진행중)

## 기능

### Step1

[Open Weather Map API](https://openweathermap.org/)를 사용하여 현재 날씨와 5일 예보 데이터를 받아왔다.  
JSON 데이터를 활용가능한 모델 타입을 구현했다.  
재사용성을 고려해서 네트워킹 타입을 구현했으며, 라이브러리 없이 URLSession을 활용했다.

</br>

## 새롭게 알게된 내용

### APIKey를 hiding하는 방법
기존의 static 상수로 표시하던 APIKey 문자열을 외부 저장소에 올려 공유되는 형태를 방지하는 방식에 대해 접하고 적용해보았다.  
현재 사용하는 APIKey를 이용해서 민감정보가 유출될 큰 위험성은 없지만, 추후에 민감한 정보가 유출될 위험이 있는 APIKey를 다루게 될 때 필요할 것 같아 공부했다.

> **참고자료**  
> [애플리케이션의 민감한 정보를 보호하는 방법 nshispter.co.kr](https://nshipster.co.kr/secrets/)  
> [Fetching API Keys from Property List Files](https://peterfriese.dev/reading-api-keys-from-plist-files/)


### Codingkeys에 이미 적용된 `convertFromSnakeCase` 속성   
- `keyDecodingStrategy`의 `convertFromSnakeCase`를 사용하여 Snake-case로 넘어오는 key값을 Camel-case 이름으로 변환되도록 구현했다.  
- JSON 데이터의 Key값 `temp_min`이 `tempMin`으로 변환되어 있었다.  
```swift
private enum Codingkeys: String, CodingKey {
    // ...
    case minimumTemperature = "tempMin"
    // 예상 헸던 코드
    // case minimumTemperature = "temp_min"
}
```

</br>

## 고민한 내용  

### Decoderable 모델 구현 관련  

- 전체 프로퍼티를 옵셔널로 만들어 보았다:   
    - API에서 제공하는 값들이 필수로 제공된다는 것을 보장하지 않았다. 데이터가 넘어오지 않을 것을 대비해 옵셔널로 선언했다.  


### SwiftLint exclude 관련  
❓`AppDelegate.swift`, `SceneDelegate` 파일을 `exclude` 할까?  
`exclude` 하는 이유에 대해 추측했다.  
- 가정1: 앱에 중요한 비즈니스 로직이 들어가지 않는다.  
- 가정2: 앱 프로젝트의 공통의 영역이므로 파트별로 컨벤션이 다를 수 있는 영역이다.  

그럼에도 컨벤션을 유지할 수 있는 장점이 있어서 이번 프로젝트에서 `exclude`에 포함하지 않았다.  


### URLSessionDataTask의 초기화 문제
![](https://i.imgur.com/MgZrmnk.png)  
- `URLSessionDataTask`를 상속한 `MockURLSessionDataTask`의 init()이 iOS 13부터 deprecated 되기 때문에 다른 방식으로 구현을 고민했다.
- `URLSessionDataTaskProtocol` 을 정의하고, `URLSessionDataTask`가 프로토콜을 채택하도록 바꿨다.
- 새롭게 변형된 반환값을 맞추기위해 `makeCustomDataTask(_:_:) -> URLSessionDataTaskProtocol` 메서드를 `URLSessionProtocol`로 주입했다.


### API를 생성해주는 Enum 타입에서 타입프로퍼티 초기화 고민
- APIKey를 .plist의 형태로 코드가 아닌 문서로 저장하는 형태의 구현을 했는데. Enum 타입의 타입 프로퍼티로 읽어오는 과정을 고민해봤다.
```swift
private static var apiKey: String {
        // .. 파일 읽어오기
        return apiKey }
private static let appID = WeatherAPI.apiKey
```
- 실험을 통해 `appID` 호출은 최초 호출에만 연산프로퍼티가 동작하고 이후에는 저장된 값을 읽어오는 걸 확인 후 해당 방식으로 구현했다.


### UnitTest XCTestCase의 프로퍼티 초기화 고민
```swift
class DecodingTests: XCTestCase {
     var currentSut: CurrentData!
}
```
- 일반적으로 위 코드처럼 초기화 하고 `setUp()` 또는 `setUpWithError()` 메서드 재정의에서 위 프로퍼티를 초기화 하는 형태를 가지는데 왜 그렇게 하는지 또는 다른 초기화 방법은 없는지 고민해보았다.  해당 [PR Review Comment](https://github.com/yagom-academy/ios-weather-forecast/pull/40/files/a0cc11e3595da14ec4fe7c561162b715f5d80153#r721846246)와 답변 


## 학습 키워드
- Apple Developer
    - [URLSession](https://developer.apple.com/documentation/foundation/urlsession/)
    - [URLComponents](https://developer.apple.com/documentation/foundation/urlcomponents/)
    - [JSONDecoder.KeyDecodingStrategy.convertFromSnakeCase](https://developer.apple.com/documentation/foundation/jsondecoder/keydecodingstrategy/convertfromsnakecase)
    - [Understanding Setup and Teardown for Test Methods](https://developer.apple.com/documentation/xctest/xctestcase/understanding_setup_and_teardown_for_test_methods)
- Swift 
    - [Enumeration](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html)

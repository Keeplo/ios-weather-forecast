## iOS 커리어 스타터 캠프

# WeatherForecast
## Information
* 프로젝트 기간 : 2021.09.27. ~ 2021.10.22.
* 프로젝트 인원 : 개인 Marco(@Keeplo)
* 프로젝트 소개 
    > Open API 를 이용해서 선택 지역 및 현재 위치의 날씨를 제공하는 앱
* Pull Requests
    * [Step 2](https://github.com/yagom-academy/ios-weather-forecast/pull/48)  
    * [Step 3](https://github.com/yagom-academy/ios-weather-forecast/pull/61)
    * [Step 4](https://github.com/yagom-academy/ios-weather-forecast/pull/67)

### Tech Stack
* Swift 5.4
* Xcode 12.5
* iOS 14.0

## Demo
<details><summary>GIF</summary><div markdown="1">
    
![137514536-144b51be-9dae-4f24-bdb3-6a5e3f991808](https://user-images.githubusercontent.com/24707229/160289197-7466e093-a78f-4ee2-9a14-cfcd284f05b1.gif)
![137514516-ee34a729-3745-40a2-89e3-67f9efbac0a8](https://user-images.githubusercontent.com/24707229/160289204-0df18134-3ec0-4b01-8ad9-730bb186fccf.gif)
</div></details>

## 고민했던 점
<details><summary>3가지 비동기 요청을 하나의 태스크로 처리하기</summary><div markdown="1">

앱에서 현 좌표를 이용해서 주소정보, 오늘날씨, 5일날씨를 각각따로 요청하고 3가지 정보를 한번에 화면에 그려주는 형태로 동작한다.
<details><summary>예제코드</summary><div markdown="1">

```swift
private func requestWeatherData(_ location: CLLocation, completion: @escaping () -> Void) {
        DispatchQueue.global().async {
            let preparingGroup = DispatchGroup()

            preparingGroup.enter()
            self.fetchAddressInfomation(location) {
                preparingGroup.leave()
            }
            preparingGroup.enter()
            self.fetchCurrentWeatherData(location) {
                preparingGroup.leave()
            }
            preparingGroup.enter()
            self.fetchFiveDayWeatherData(location) {
                preparingGroup.leave()
            }

            preparingGroup.wait()
            completion()
        }
    }
```
</div></details>

> global 비동기 태스크를 만들어 DispatchGroup으로 3가지 비동기 태스크를 관리해서 해결했다.

</div></details>
<details><summary>MockURLSession 객체를 이용해서 URLSession 동작 UnitTest</summary><div markdown="1">
    
```swift
protocol URLSessionProtocol {
    func makeCustomDataTask(with url: URL,
                            completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void)
    -> URLSessionDataTaskProtocol
}

extension URLSession: URLSessionProtocol {
    func makeCustomDataTask(with url: URL,
                            completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void)
    -> URLSessionDataTaskProtocol {
        return dataTask(with: url, completionHandler: completionHandler) as URLSessionDataTaskProtocol
    }
}

protocol URLSessionDataTaskProtocol {
    func resume()
}

extension URLSessionDataTask: URLSessionDataTaskProtocol {}
```
URLSession에서 사용할 동작을 Protocol을 이용해서 추상화하고
```swift
struct MockURLSessionDataTask: URLSessionDataTaskProtocol {
    
    var resumeDidCall: () -> Void = {}
    
    func resume() {
        resumeDidCall()
    }
}

struct MockURLSession: URLSessionProtocol {
    
    private let isSuccess: Bool
    
    init(isSuccess: Bool) {
        self.isSuccess = isSuccess
    }
    
    func makeCustomDataTask(with url: URL,
                            completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void)
    -> URLSessionDataTaskProtocol {
        let successResponse = HTTPURLResponse(url: url,
                                              statusCode: 200,
                                              httpVersion: nil,
                                              headerFields: nil)
        let failureResponse = HTTPURLResponse(url: url,
                                              statusCode: 400,
                                              httpVersion: nil,
                                              headerFields: nil)
        
        var mockURLSessionDataTask = MockURLSessionDataTask()
        mockURLSessionDataTask.resumeDidCall = {
            completionHandler(Data(), self.isSuccess ? successResponse : failureResponse, nil)
        }
        
        return mockURLSessionDataTask
    }
}
```
다음과 같이 Mock 객체르 만들어서 테스트르 진행함
```swift
class MockURLSessionTests: XCTestCase {
    
    var successSut: URLSessionProtocol!
    var failureSut: URLSessionProtocol!

    override func setUpWithError() throws {
        super.setUp()
        successSut = MockURLSession(isSuccess: true)
        failureSut = MockURLSession(isSuccess: false)
    }

    override func tearDownWithError() throws {
        successSut = nil
        failureSut = nil
        super.tearDown()
    }

    func test_SuccessCase_dataTask메서드로_통신성공하면_statusCode가_200이상300미만이다() {
        // given
        let successRange = 200..<300
        let url: URL! = URL(string: "https://yagom.com")
        // when
        successSut.makeCustomDataTask(with: url) { _, response, _ in
            guard let response = response as? HTTPURLResponse else {
                XCTFail("Response error.")
                return
            }
            // then
            XCTAssert(successRange ~= response.statusCode)
        }.resume()
    }
    
    func test_FailureCase_dataTask메서드로_통신실패하면_statusCode가_200이상300미만이아니다() {
        // given
        let successRange = 200..<300
        let url: URL! = URL(string: "https://yagom.com")
        // when
        failureSut.makeCustomDataTask(with: url) { _, response, _ in
            guard let response = response as? HTTPURLResponse else {
                XCTFail("Response error.")
                return
            }
            // then
            XCTAssert(!(successRange ~= response.statusCode))
        }.resume()
    }
}

```
</div></details>
<details><summary>APIKey를 Enum 타입에서 타입프로퍼티 초기화 고민</summary><div markdown="1">

* APIKey를 .plist의 형태로 코드가 아닌 문서로 저장하는 형태의 구현을 했는데. Enum 타입의 타입 프로퍼티로 읽어오는 과정을 고민해봤다.
```swift
private static var apiKey: String {
        // .. 파일 읽어오기
        return apiKey }
private static let appID = WeatherAPI.apiKey
```
> 실험을 통해 appID 호출은 최초 호출에만 연산프로퍼티가 동작하고 이후에는 저장된 값을 읽어오는 걸 확인 후 해당 방식으로 구현했다.
</div></details>

<br>

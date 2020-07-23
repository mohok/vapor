몇달 전 백엔드를 맡아 달라는 회사의 요청이 있었습니다. 기존 서버는 Python/Django로 구축되어 있었고 저는 아무것도 모르는 상황이었습니다. 그래서 주류 언어(Python) + 주류 프레임워크(Django)를 학습하는 일보다 비주류 프레임워크(Vapor)만을 학습하는 일이 편하리라 생각하면서 Vapor 4-beta를 시작하게 되었습니다.

그것은 끔찍한 생각이었습니다. 구현되지 않은 기능이 많았음은 물론이고, 버그에 가까운 동작도 많았습니다. 프레임워크의 문제인지, 작성한 코드의 문제인지 파악하기 어려웠습니다. 그럴 때마다 디스코드의 vapor 채널이 많은 도움이 되었습니다. 현재 vapor 4(이하 vapor)는 공식으로 릴리즈되었습니다. 또, 회사 서비스의 일부분을 담당하고 있습니다. 기존 서버의 API 쿼리 튜닝까지 vapor에서 병행하면서 전체에서 담당하는 비율은 높아지는 중입니다. 

누군가 vapor로 서비스의 메인 서버를 구축하겠다면 선뜻 추천하지는 않을 것입니다. 하지만, 사이드 프로젝트 정도라면, 게다가 클라이언트 개발자로서 서버 프레임워크를 간단히 맛보고 싶은 정도라면 충분히 매력적이라고 생각합니다. 

몇 가지 간단한 튜토리얼을 공유하고자 합니다. 시작에 도움이 되었으면 좋겠습니다.

# 목차

1. Vapor 4 시작하기
2. Fluent 4 시작하기
3. CRUD API 만들기
4. Ubuntu에서 배포하기
5. iOS + Vapor
6. Authentication
7. ...


# 1. Vapor 4 시작하기

* Swift 설치
* Vapor Toolbox 설치
* 새 프로젝트 생성
* 프로젝트 실행
* RouteCollection

## Swift 설치

Vapor를 사용하기 위해서*는 Swift 5.2 이상 버전이 필요합니다. 터미널을 실행하고 다음과 같이 입력하여 버전을 확인할 수 있습니다. Swift 버전이 낮다면 Xcode를 최신 버전으로 업데이트하거나 직접 [패키지를 다운](https://swift.org/download/#releases)받아 설치합니다.

```
swift --version
// Apple Swift version 5.2.4 (swiftlang-1103.0.32.9 clang-1103.0.32.53)
// Target: x86_64-apple-darwin19.4.0
```

 * [macOS의 경우 10.15 이상](https://forums.swift.org/t/announcement-vapor-4-will-no-longer-support-macos-10-14/33511)

## Vapor Toolbox 설치

다음으로는 brew를 이용해 vapor toolbox를 설치합니다. Toolbox는 터미널에서 vapor와 관련된 커맨드를 입력받을 수 있게 만들어 줍니다. 설치 후 `vapor --help`를 입력하면 커맨드를 확인할 수 있습니다.

```
brew install vapor/tap/vapor
```

## 새 프로젝트 생성

이번 포스트에서는 간단한 프로젝트를 만들어 보겠습니다. 프로젝트 이름은 hello로 하겠습니다.

```
// vapor new <#project name#>
vapor new hello
```

```
Cloning template...
name: hello
Would you like to use Fluent?
y/n> 
```

Fluent는 Vapor를 위한 ORM 프레임워크 입니다. 아직 데이터베이스를 사용하지 않을 것이므로 n을 입력합니다.

현재 SQLite, MySQL(MariaDB), PostgreSQL, MongoDB 드라이버(+ Redis)가 지원되고 있습니다. 각 데이터베이스 드라이버는 프로젝트가 생성된 이후에도 SPM을 이용해 추가할 수 있습니다.

## 프로젝트 실행

방금 생성한 프로젝트를 Xcode를 이용해 열어 보겠습니다. 프로젝트 디렉토리에서 Toolbox 커맨드를 입력하거나 Package.swift 파일을 실행합니다.

```
cd hello
vapor xcode 
```

실행 스킴을 hello, 디바이스를 My Mac으로 설정하고 실행 버튼을 눌러 보겠습니다. 

```
[ NOTICE ] Server starting on http://127.0.0.1:8080
```

디버그 콘솔에 서버 실행 메시지가 출력되었습니다. 웹 브라우저를 열고 [http://127.0.0.1:8080](http://127.0.0.1:8080) 를 입력하여 실행하겠습니다.

```
It works!
```

페이지를 확인했다면 다시 Xcode로 돌아옵니다. 디버그 콘솔을 보면 우리가 접속했던 로그가 있습니다(Vapor의 로깅 시스템 [SwiftLog](https://github.com/apple/swift-log)에 기반하고 있습니다).

```
[ INFO ] GET /
```

HTTP 요청에 대한 로그는 기본적으로 [ `Log Level` ] `HTTP Method`  /`URI` 형식으로 표기됩니다. 

이번에는 웹 브라우저에 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) 를 입력하고 다시 Xcode로 돌아오면 아래와 같은 로그가 확인됩니다.

```
[ INFO ] GET /hello
```


이 두 경로의 요청에 대한 처리는 Sources/App/routes.swift 에서 확인할 수 있습니다.  HTTP get 메소드 `/`,  `/hello` 로 오는 응답에 대한 처리가 정의되어 있습니다. 

```swift
// routes.swift

import Vapor

func routes(_ app: Application) throws {
  app.get { req in
    return "It works!"
  }
  
  app.get("hello") { req -> String in
    return "Hello, world!"
  }
}
```

라우터 메소드를 클로저로 표현할 수도 있지만 범용적인 사용을 위한 다른 방법도 제공됩니다. 그 전에, 툴박스를 통해 생성된 프로젝트 구조를 잠깐 살펴 보겠습니다. 

서버를 실행하면 Sources/Run/main.swift 파일이 먼저 실행됩니다.  중간에 `try configure(app)` 을 따라가 보겠습니다.

```swift
// main.swift
import App
import Vapor

var env = try Environment.detect() // 환경 변수를 불러옵니다.
try LoggingSystem.bootstrap(from: &env) // 로깅 시스템을 시작합니다(SwiftLog).
let app = Application(env) // 프로그램의 라이프사이과 기능을 관리하는 객체
defer { app.shutdown() }
try configure(app) // <<--
try app.run()
```

Sources/App/configure.swift을 열어보면, `routes` 메소드가 실행되고 있습니다. `configure` 메소드를 꼭 사용해야 하는 것은 아니지만, 일반적으로 이곳에서 Application 객체를 활용합니다.  다시  `try routes(app)` 메소드를 따라가 routes.swift 파일로 이동해 보겠습니다.

```swift
// configure.swift
import Vapor

// configures your application
public func configure(_ app: Application) throws {
  // uncomment to serve files from /Public folder
  // app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))
  
  // register routes
  try routes(app)
}
```

몇 가지 HTTP 메소드를 더 만들고 Application 객체에 등록해 보겠습니다.

```swift
import Vapor

func routes(_ app: Application) throws {
  app.get { req in
    return "It works!"
  }
  
  app.get("hello") { req -> String in
    return "Hello, world!"
  }
  
  // 1
  app.get("hello", ":name") { req -> String in
    let name = req.parameters.get("name")!
    return "Hello, \(name)"
  }
  
  // 2
  app.post("hello") { req -> HTTPStatus in
    return HTTPStatus.unauthorized
  }
  
  // 3
  struct Todo: Content {
    let id: UUID?
    let title: String
    let content: String
    let createdAt: Date?
  }

  // 4
  app.post("todo") { req -> Todo in
    let body = try req.content.decode(Todo.self)
    return Todo(
        id: UUID(),
        title: body.title,
        content: body.content,
        createdAt: Date()
    )
  }
  
  
  /*
   // 1
   curl "http://127.0.0.1:8080/hello/teemo"

   // 2
   curl -X "POST" "http://127.0.0.1:8080/hello"
   
   // 4
   curl -X "POST" "http://127.0.0.1:8080/todo" \
   -H 'Content-Type: application/json; charset=utf-8' \
   -d $'{
   "title": "밥먹기",
   "content": "삼겹살 3인분"
   }'

   */
}
```

req는 [Request](https://github.com/vapor/vapor/blob/master/Sources/Vapor/Request/Request.swift)의 줄임말 입니다. 이 객체는 HTTP 요청을 처리합니다. Curl을 활용해 새로 정의한 API를 요청해 보겠습니다.

1. 파라미터 경로를 `:<#parameter#>` 형식으로 정의했습니다. 
2. HTTP POST 메소드도 쉽게 등록할 수 있습니다. 요청에 대한 응답 또한 사용자가 정의하는 타입으로 반환됩니다.
3. Request 객체를 통한 요청과 응답은 [Content](https://github.com/vapor/vapor/blob/master/Sources/Vapor/Content/Content.swift) 프로토콜을 채택해야 합니다. 이 프로토콜은 Codable, RequestDecodable, ResponseEncodable을 채택하고 있습니다. RequestDecodable, ResponseEncodable 프로토콜은 비동기 처리를 위한 프로토콜입니다. 
4. Content를 채택한 타입에 대한 요청은 Request 객체에 의해 [Media Type](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Type)에 따라 디코딩될 수 있습니다. 객체를 인코딩하여 응답하는 일 또한 가능합니다.

앞서 HTTP 메소드와 경로를 설정하여 Application 객체에 등록하고, 이 요청에 대한 응답까지 알아봤습니다.

생각해 봅시다. 하나의 프로젝트에 많은 API가 필요할텐데 어떻게 관리해야 할까요? routes 메소드 안에 모두 정의해야 할까요? 이 문제를 해결하기 위해 RoutesBuilder 프로토콜을 살펴보겠습니다.

프로젝트에 Sources/Controller/HelloController.swfit 파일을 하나 생성하겠습니다. 그리고 RouteCollection 프로토콜을 채택하겠습니다.

```swift
// HelloController.swfit.swift

import Vapor

final class HelloController.swfit: RouteCollection {
  func boot(routes: RoutesBuilder) throws {
  
  }
}
```

새로운 프로토콜 `RouteCollection`과 `RoutesBuilder`가 등장했습니다. 사실 RoutesBuilder는 `app.get("hello") { req -> String in` 와 같이 HTTP 메소드를 Application 객체에 등록할 때 이미 사용하고 있던 프로토콜입니다.  이 프로토콜의 `grouped` 메소드를 사용하면 경로를 그룹으로 관리하거나, 경로에 대한 미들웨어를 한번에 관리할 수 있습니다.

미들웨어는 광범위하게 쓰이는 용어인데, 쉽게 이야기해서 네트워크로 요청을 처리하기 전에 다른 일을 처리하게 만들 수 있습니다.. 예를 들어서, '아! 회원가입을 할 때만 특별한 로그를 남기고 싶다.'는 생각이 들면 RoutesBuilder를 이용해 미들웨어를 하나 등록해 주면 됩니다.

```swift
let logged = app.groupted(특별한로그미들웨어())
logged.post("회원가입") { ...
```

RoutesBuilder는 경로도 그룹으로 만들 수 있습니다.

```swift
let v1 = app.grouped("v1")
v1.get("hello") { ...

let auth = v1.grouped("auth")
auth.post("sign-in") { ...

// GET: /v1/hello
// POST: /v1/auth/sign-in
```

HelloController.swift 파일로 돌아가 RouteBuilder를 사용해 보겠습니다.


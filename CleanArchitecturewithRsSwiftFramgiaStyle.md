# Clean Architecture with RxSwift - Framgia Style
Chúng ta đều biết về các level sẽ trông như thế này:

![level](https://viblo.asia/uploads/dd700b20-877e-4fe0-bf46-24e0092f8f92.png)

Và các level được giải thích như sau:
### Domain

Domain là cơ bản ứng dụng của bạn là gì và nó có thể làm gì (Entities, UseCase, vv..) Nó không phụ thuộc vào UIKit hay bất cứ khuôn khổ nào cả, và nó không có sự implement ngoài các Entities.
### Platform

Platform là việc implement cụ thể của Domain trong một nền tảng cụ thể như iOS. Nó giấu tất cả các chi tiết được implement. Ví dụ việc implement database cho dù đó là CoreData, Realm, SQLite, vv..
### Application

Application có trách nhiệm cung cấp thông tin cho người dùng và xử lý đầu vào của người dùng. Nó có thể được thực hiện với bất kỳ pattern nào như MVVM, MVC, MVP. Đây là nơi dành cho tất cả UIView và UIViewController của bạn. Như bạn thấy từ ứng dụng ví dụ, tất cả ViewController hoàn toàn độc lập với Platform. Trách nhiệm duy nhất của ViewController là bind UI với Domain để kết nối mọi thứ.

> Tuy nhiên, bạn có thể thắc mắc rằng: rõ ràng UseCase phụ thuộc vào đặc tính của màn hình: tức là các thao tác của người dùng trên màn hình, có thể làm gì ở màn hình đó,... => UseCase lẽ ra phải thuộc về lớp `Application` chứ?
> Tuy vậy, code Usecase được viết từ trước so với code của view, nó định nghĩa sẵn các công việc rồi, phần view chỉ implement nó ra thôi, nó chính nằm ở câu định nghĩa Domain ở trên \*Domain là cơ bản ứng dụng của bạn là gì và nó có thể làm gì\*
> Nhưng vì liên quan đến mỗi màn hình/tính năng nên file UseCase thường được đặt chung chỗ với các file Controller, viewModel và Navigator

## Thứ tự viết code cho 1 màn hình/chức năng cụ thể là như thế nào?
Dĩ nhiên chúng ta sẽ viết code theo chiều từ trong ra ngoài (xét theo hình minh hoạ về 3 layer ở trên) nhưng hơi khác chút =)). Tức là: Domain ->  Application -> Platform
Tức là:
- Entities-> Usecase -> ViewModel (Navigator,..) -> ViewController
- UnitTest
- Xong khi đảm bảo phần logic đã chạy đúng thì mới lắp API, giao diện vào,...Platform (và đây là phần platform) API, View,...

## Viết code cho phần Domain
 Domain bao gồm phần Entities và UseCases. Hãy nhìn vào ví dụ [github](https://github.com/tuan188/MGCleanArchitecture)
Chúng ta hãy cùng nhìn vào màn hình `Repo List`  mà cụ thể là `ReposViewController` 
![Product](https://gyazo.com/ff353252653cdd8d88bc7498f80541d8.png)
Phần code của Entity Repo như sau:

```
import ObjectMapper

struct Repo {
var id = 0
var name: String
var fullname: String
var urlString: String
var starCount: Int
var folkCount: Int
var avatarURLString: String
}

extension Repo {
init() {
self.init(
id: 0,
name: "",
fullname: "",
urlString: "",
starCount: 0,
folkCount: 0,
avatarURLString: ""
)
}
}

extension Repo: Then, HasID, Hashable { }

extension Repo: Mappable {

init?(map: Map) {
self.init()
}

mutating func mapping(map: Map) {
id <- map["id"]
name <- map["name"]
fullname <- map["full_name"]
urlString <- map["html_url"]
starCount <- map["stargazers_count"]
folkCount <- map["forks"]
avatarURLString <- map["owner.avatar_url"]
}
}
```
Ở đây sử dụng `struct` thay vì `class` bởi vì `class` là kiểu tham chiếu trong khi `struct` là kiểu tham trị tức là giá trị của nó được `copy` khi gán tới 1 biến hoặc truyền như parametter trong function. 
Tiếp theo bạn sẽ thắc mắc về việc `Repo` phải conform lại những protocol `Then`, `HasID` và `Hashable`
thì `Then` là cocoaPod để
```
/// Makes it available to set properties with closures just after initializing and copying the value types.
///
///     let frame = CGRect().with {
///       $0.origin.x = 10
///       $0.size.width = 100
///     }
```
Mà cụ thể ta sẽ thấy trong `UseCase` của repo:
```
var getRepoList_ReturnValue: Observable<PagingInfo<Repo>> = {
let items = [
Repo().with { $0.id = 1 }
]
let page = PagingInfo<Repo>(page: 1, items: OrderedSet(sequence: items))
return Observable.just(page)
}()
```
Còn protocol `HasID` là 1 protocol tự viết để  quản lý việc conform protocol `Hashable`
```
protocol HasID {
associatedtype IDType
var id: IDType { get }
}

extension Hashable where Self: HasID, Self.IDType == Int {
var hashValue: Int {
return id
}

static func == (lhs: Self, rhs: Self) -> Bool {
return lhs.id == rhs.id
}
}

extension Hashable where Self: HasID, Self.IDType == String {
var hashValue: Int {
return id.hashValue
}

static func == (lhs: Self, rhs: Self) -> Bool {
return lhs.id == rhs.id
}
}

```

Trong đó việc 1 entity của chúng ta như `Repo` phải conform protocol `Hashable` là để entity `Repo` có thể sử dụng nó với `OrderedSet` để sắp xếp.

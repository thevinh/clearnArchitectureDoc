### UseCase
Sau khi viết code cho các Entities, theo đúng thứ tự như [phần I](https://github.com/thevinh/clearnArchitectureDoc/blob/master/CleanArchitecturewithRsSwiftFramgiaStyle.md) tiếp theo ta sẽ viết code cho phần UseCase.
Như đã phân tích ở phần trước: file UseCase được đặt cùng chỗ với các file thuộc lớp `Application` như: Controller, ViewModel, Navigator,... cho dù nó thuộc về Domain.
Bạn hãy tìm file `ReposUseCase` ở thư mục sau: Scenes/Repos

```
protocol ReposUseCaseType {
func getRepoList() -> Observable<PagingInfo<Repo>>
func loadMoreRepoList(page: Int) -> Observable<PagingInfo<Repo>>
}

struct ReposUseCase: ReposUseCaseType {
let repository: RepoRepositoryType

func getRepoList() -> Observable<PagingInfo<Repo>> {
return loadMoreRepoList(page: 1)
}

func loadMoreRepoList(page: Int) -> Observable<PagingInfo<Repo>> {
return repository.getRepoList(page: page, perPage: 10)
}
}
```

Cấu trúc của file UseCase bao gồm 2 phần: 1 protocol và 1 struct conform lại protocol ấy. Trong đó Protocol định nghĩa các UseCase, còn Struct thì implement các UseCase cụ thể ấy
Cái này được gọi là `Dependency Inversion` chính là nguyên lý cuối cùng (chữ D) của `SOLID`. Các bạn có thể đọc thêm ở [Đây][https://toidicodedao.com/2015/11/03/dependency-injection-va-inversion-of-control-phan-1-dinh-nghia/]
Nhưng có thể hiểu đơn giản là Các Class giao tiếp với nhau thông qua interface (ở đây là Protocol) chứ không phải thông qua Implementation để đảm bảo việc khi module cấp thấp thay đổi, các module cấp cao không phải thay đổi theo, giúp việc bảo trì, thay đổi hay testing dễ dàng hơn.
Khoan hẵng bàn về code `RxSwift` thì chúng ta có thể thấy code rất dễ đọc. Có thể diễn giải ra rằng: Có 2 usecase ứng với Repo là: lấy danh sách các repo (`getRepoList`) và load thêm các repo (`loadMoreRepoList`) với đầu vào là số trang hiện tại. Trong đó hàm `getRepoList` thực chất chính là `loadMoreRepoList` với page bằng 1. Bên trong method `loadMoreRepoList` gọi tới method của `RepoRepository`

```
protocol RepoRepositoryType {
func getRepoList(page: Int, perPage: Int, useCache: Bool) -> Observable<PagingInfo<Repo>>
}
```
Về cơ bản nó vẫn chỉ là protocol, còn code của nó đầy đủ về việc get repo list về như thế nào thì sẽ được viết ở dưới như trong file:

```
final class RepoRepository: RepoRepositoryType {
func getRepoList(page: Int, perPage: Int, useCache: Bool) -> Observable<PagingInfo<Repo>> {
let input = API.GetRepoListInput(page: page, perPage: perPage)
input.useCache = useCache
let output: Observable<API.GetRepoListOutput> = API.shared.request(input)
return output.map { output in
guard let repos = output.repos else {
throw APIInvalidResponseError()
}
return PagingInfo<Repo>(page: page, items: repos)
}
}
}
```
Phần code phia trên thuộc về phần platform.
Tóm lại phần `UseCase` có thể coi như là phần phác hoạ ra trước, mang tính trừu tượng là nhiều, code cụ thể sẽ được viết ở các phần sau.

### ViewModel
Bài viết này đang viết ở thời điểm commit code: `d3d8658 - (2 days ago) Update MGArchitecture pod`
Bạn sẽ thấy bên trong folder  `Scenes/Repos` có tới 2 file `RepoViewModel` và `ReposViewModel` chỉ khác nhau một chữ s. Đơn giản thì file `RepoViewModel` chỉ như một file helper của model `Repo` mà thôi

```
import UIKit

struct RepoViewModel {
let repo: Repo

var name: String {
return repo.name
}

var url: URL? {
return URL(string: repo.avatarURLString)
}
}
```
Còn thứ mang ý nghĩa viewModel thực sự là `ReposViewModel`


// Note:
Về hệ thống ViewController, ViewModel và Navigator thì
Navigator là 1 struct có chứa 1 property kiểu UINavigationController thường được đặt tên là `navController`
`navController` này được pass qua các VIewController bằng code:
```
let navigator = FilePreviewNavigator(navigationController: navigationController)
```
Bởi vì Navigator là 1 struct (do cách đặt tên nên thường bị hiểu nhầm là 1 NavigationController) nên nó có 1 Constructor mặc định để khởi tạo mọi property, các VIewController dĩ nhiên vẫn chỉ dùng chung 1 navigationcontroller được pass qua nhau để có thể push thêm view mới vào như code:
```
navigationController.pushViewController(vc, animated: true)
```


Dependency Injection: https://toidicodedao.com/2015/11/03/dependency-injection-va-inversion-of-control-phan-1-dinh-nghia/

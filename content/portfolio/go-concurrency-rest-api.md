---
title: "go concurrency rest api"
date: 2017-09-10T15:45:42+07:00
draft: true
image: "img/portfolio/gophermegaphones.jpg"
---

Đầu năm 2016, tôi và các cộng sự ở công ty đã rất hứng khởi và thích thú với tìm hiểu và ứng dụng `Golang` cho các dự án mới.
Các lập trình viên khi tiếp cận với `Golang` thì đều cảm giác rất thân quen và dễ dàng có thể code nhoay nhoáy trong vòng 2-3 ngày hay đến 1 tuần là cùng.
Bởi `Golang` được thừa kế từ ngôn ngữ lập trình `C` mà ngôn ngữ lập trình `C` thì chắc lập trình viên nào cũng được học và sử dụng trong trường đại học cả rồi. 
Đây cũng là ý kiến chủ quan của một lập trình viên tay ngang và thuộc thế hệ 8x như tôi. Ngôn ngữ `C/C++` thì bản thân tôi cũng chưa được học bao giờ. 
Chẳng là hồi học đại học năm thứ nhất đại học, tôi vẫn tiếp tục hì hục ngồi code các bài toán trên ngôn ngữ lập trình `Pascal`, ngôn ngữ lập trình mà tôi đã được học từ năm lớp 10 ở trường PTTH, thằng bạn cùng lớp thấy thế nên nó bảo tôi: "Nếu mày muốn thành lập trình viên chuyên nghiệp thì kiểu gì mày cũng phải học `C/C++`".
Nên tôi và nó có mò lên hiệu sách trên phố sách đình đám nhất Hà Thành, *Tràng Tiền*, lê la khắp các hiệu sách để tìm mua 1 quyển lập trình `C++` và sau đó có mua thêm một quyển lập trình `Assembly` của *Peter Norton* khá nổi tiếng và phổ biến những năm đó thì phải.
Tôi vẫn nhớ như in là tôi đã rất ấn tượng với bìa sách hình chân dung ở tư thế đang khoanh tay trước ngực của Peter Norton. Một phong thái tự tin và rất thông minh.

Thời của tôi là vậy, các thế hệ Y sau này chắc sẽ không còn học C nữa, mà có thể là `Haskell` hay một ngôn ngữ nào mới mẻ khác mà chúng ta chưa biết.
Các ngôn ngữ lập trình hiện đại thì ngày càng dễ dàng cho lập trình viên hơn. Các lập trình viên ở *Google* cho ra đời `Golang` chắc cũng nhằm mục đích đơn giản hoá việc lập trình trên ngôn ngữ `C` mà vẫn đạt được hiệu năng cao.

Hôm nay, tôi làm một code mẫu tạo REST API đơn giản bằng Golang để thấy được việc lập trình *concurrency* bằng Golang nó trở nên đơn giản đến mức nào nếu so sánh với việc làm tương tự trên ngôn ngữ lập trình C/C++ hồi xưa.
Tôi xin nhấn mạnh là `C/C++` của thời *ông bà anh* nhé vì bản thân `C/C++` vẫn đang phát triển và mới cho ra phiên bản năm 2017 có nhiều điểm mới mà tôi cũng không biết cụ thể nó là cái gì? :)
Và ở một cách tiếp cận khác, gần đây các ngôn ngữ lập trình đều có thư viện `reactive programming` cũng khá thuận tiện cho việc xử lý các yêu cầu asynchronous nhằm tăng độ response của các ứng dụng.
Tuy nhiên, tuỳ vào yêu cầu bài toán và cách thiết kế hệ thống mà chúng ta cần lựa chọn sử dụng cách tiếp cận nào cho phù hợp và dễ dàng hơn.

*Yêu cầu* của bài toán là viết một API trả về nhiệt độ trung bình từ nhiều nguồn số liệu của một thành phố.
Thông thường, nếu sử dụng các lập trình tuần tự thì chúng ra phải lấy nhiệt độ của thành phố của từng nguồn cung cấp rồi tính ra được nhiệt độ trung bình.
Giả dụ chúng ta có 10 nguồn cung cấp số liệu và để lấy nhiệt độ của mỗi nguồn mất 1 giây thì tổng thời gian cho việc lấy nhiệt độ trung bình của API sẽ là 10 giây.
Opsss, 10 giây, một thời gian khó có thể chấp nhận cho một API trả về kết quả.

Giải pháp để tăng hiệu năng cho API ở đây là sử dụng lập trình `go concurrnecy` kết hợp với `channel` để có thể cùng một lúc lấy được nhiệt độ của các nguồn cung cấp trong cùng một thời điểm.
Kết quả là chúng ra chỉ cần, tương đối, 1 giây để có thể lấy được kết quả từ 10 hay nhiều nguồn số liệu khác nhau. Một sự khác biệt đáng kinh ngạc?

Cụ thể các bước tiến hành cài đặt như sau.

* Trước tiên thì chúng ta định nghĩa một interface chung để lấy nhiệt độ của một thành phố từ các nguồn số liệu khác nhau

{{< gist 6c9af258613ef678527a86ebe70f8df5 >}} 

* Vì mỗi nguồn số liệu có cách lấy số liệu khác nhau nên chúng ta cần khai báo các `struct` tương ứng để cài đặt. 
Ở đây để đơn giản thì tôi lấy từ 2 nguồn số liệu là `OpenWeatherMap` và `apixu`

{{< gist 72ebbfbc37782f4bd431f76e739c1ed2 >}}

{{< gist cf89c337ea74c74117fd92cd1b1af4cf >}}

* Sau khi, đã có thể lấy được số liệu từ các nguồn cụ thể thì chúng ta sẽ sử dụng `go concurrency` và `channel` để cài đặt cách thức để lấy số liệu của các nguồn được diễn ra cùng một lúc để giảm thiểu thời gian chờ đợi.

{{< gist 0d357f31d8b4b570c539a7f8a8819533 >}}

Ở đây, các bạn có thể thấy việc gọi một method concurrency bằng Golang khá đơn giản, chỉ cần dùng từ khoá `go` trước lời gọi hàm mà thôi.

```
go func(p weatherProvider) {
```

Mỗi lời gọi này thì Go sẽ tạo riêng một `goroutine` chạy độc lập nhau nên việc lấy số liệu ở từng nguồn được thực hiện gần như đồng thời.
Và các `goroutine` này sẽ giao tiếp với nhau thông qua `channel` nên chúng ta cũng định nghĩa một channel chung để có thể lưu trữ số liệu của các nguồn khác nhau ở cùng một `channel`.

```
temps := make(chan float64, len(w))
...
...
...

temps <- k
```

Cuối cùng, thì tổng hợp lại chúng ta có thể expose thành một REST API thông qua hàm `main() ` như sau

{{< gist 9d02306d1cd8472e321330ec658928aa >}}

Như vậy, chúng ta có thể dễ dàng cài đặt một REST API hiệu quả bằng cách sử dụng `go concurrency` và `channel`. 
Source code mẫu các bạn có thể xem trên github: https://github.com/hadv/go-concurrency-rest-api
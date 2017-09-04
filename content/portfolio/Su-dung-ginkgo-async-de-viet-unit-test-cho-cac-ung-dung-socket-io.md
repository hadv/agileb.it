---
title: "Sử dụng ginkgo async để viết unit test cho các ứng dụng socket.io"
date: 2017-09-04T17:45:42+07:00
draft: true
image: "img/portfolio/ginkgo.png"
---

Với sự ra đời của `websocket` thì việc giao tiếp hai chiều giữa client và server trở nên dễ dàng hơn bất kỳ lúc nào. Đặc biệt là nếu sử dụng các thư viện như `socket.io` thì việc lập trình các ứng dụng có sự giao tiếp hai chiều giữa client và server thì càng trở nên đơn giản hơn rất rất nhiều nữa. Tuy nhiên, việc thực hiện viết unit test cho việc giao tiếp qua lại giữa client và server có chút gì đó hơi khó khăn bởi vì thông tin được truyền qua lại giữa client và server không theo một tuần tự cụ thể nào cả.

Hôm nay tôi sẽ trình bày cho các bạn cách sử dụng `async test` của `Ginkgo` để viết unit test cho một ứng dụng chat đơn giản được phát triển bằng  thư viện `go-socket.io` và `Gorilla Websocket` của Golang. Hy vọng, sau khi đọc bài này các bạn có thể dễ dàng viết được unit test cho các ứng dụng **socket.io** trong Golang.

Giả dụ tôi có một ứng dụng chat đơn giản như sau. client sẽ emit một sự kiện `joined_message` lên server; sau khi server xử lý xong sẽ emit ngược lại cho client một sự kiện là `message`.

```
package main

import (
	"encoding/json"
	"github.com/googollee/go-socket.io"
	"log"
	"sync"
	"time"
)

// Should look like path
const websocketRoom = "/chat"

var connections = make(map[string]socketio.Socket)

func NewChatServer() (*socketio.Server, error) {

	sio, err := socketio.NewServer(nil)
	if err != nil {
		log.Fatal(err)
		return nil, err
	}

	sio.On("connection", func(so socketio.Socket) {
		username := so.Id()
		so.Join(websocketRoom)

		so.On("joined_message", func(message string) {
			username = message
			connections[username] = so
			log.Println("joined_message", message)
			res := map[string]interface{}{
				"username": username,
				"dateTime": time.Now().UTC().Format(time.RFC3339Nano),
				"type":     "joined_message",
			}
			jsonRes, _ := json.Marshal(res)
			so.Emit("message", string(jsonRes))
			so.BroadcastTo(websocketRoom, "message", string(jsonRes))
		})
        ...
	return sio, err
}
```

Để thực hiện viết unit test cho ứng dụng chat này chúng ta có thể làm theo các bước sau đây.

* Trước tiên, các bạn hãy cài đặt các thư viện liên quan bằng cách sử dụng lệnh `go get`

	```
	$ go get github.com/onsi/ginkgo
	$ go get github.com/onsi/ginkgo/ginkgo
	$ go get github.com/onsi/gomega
	```

* Tại thư mục dự án, các bạn chạy lệnh sau để khởi tạo _test suite_ cho ứng dụng chat.

	```
    $ ginkgo bootstrap
    ```
  Khi đó bộ `test suite` cho ứng dụng chat sẽ được tự động sinh ra trong file `gochatapp_suite_test.go` như sau

	```
    package main_test

    import (
        . "github.com/onsi/ginkgo"
        . "github.com/onsi/gomega"

        "testing"
    )

    func TestGochatapp(t *testing.T) {
        RegisterFailHandler(Fail)
        RunSpecs(t, "Gochatapp Suite")
    }
    ```

* Sau đó, chạy tiếp lệnh sau để sinh đoạn mã unit test cho ứng dụng chat

	```
    $ ginkgo generate chat
    ```
 
	Ginkgo sẽ tự sinh ra file `chat_test.go` với nội dung cơ bản như sau. Và chúng ta sẽ thêm phần code cho unit test vào bên trong phương thức **Describe()**
    
	```
    package main
  	
    import (
        . "github.com/onsi/ginkgo"
        . "github.com/onsi/gomega"
    )

    var _ = Describe("Chat", func() {
  	}
    ```

* Trước tiên chúng ra cần tạo ra một `socket.io` server để lắng nghe các sự kiện từ client gửi lên và chúng ra sẽ khởi tạo  server trong phương thức **BeforeEach()** như sau
	```
    var _ = Describe("Chat", func() {
	var server *socketio.Server

	BeforeEach(func() {
		var err error
		if server, err = NewChatServer(); err != nil {
			fmt.Println("Error", err.Error())
			return
		}
		http.Handle("/socket.io/", server)

		var listen string = os.Getenv("LISTEN")

		if listen == "" {
			listen = ":8080"
		}

		go http.ListenAndServe(listen, nil)
	})
    ```

	Các bạn có thấy là tại sao chúng ta phải sử dụng `go http.ListenAndServe(listen, nil)` không? Vâng bởi vì chúng ra chạy cả server và client trên cùng một process nên chúng ta cần sử dụng `go func()` để có thể chạy client & server song song nhau.
    
* Tiếp đến chúng ta sẽ cài đặt unit test cho sự kiện `joined_message` như sau:
	```
			JustBeforeEach(func() {
				opts := &socketio_client.Options{
					Transport: "websocket",
					Query:     make(map[string]string),
				}
				socket, err = socketio_client.NewClient("http://localhost:8080", opts)
				if err != nil {
					fmt.Println("Error", err.Error())
					return
				}
			})

			It("join chat sucessfully", func() {
				socket.Emit("joined_message", "hadv")
				c := make(chan string, 0)
				socket.On("message", func(msg string) {
					c <- msg
				})
				var dat map[string]interface{}

				if err := json.Unmarshal([]byte(<-c), &dat); err != nil {
					fmt.Println("invalid input data", err.Error())
					return
				}
				Expect(dat["type"]).To(Equal("joined_message"))
				Expect(dat["username"]).To(Equal("hadv"))
			})    
    ```
	
    Ở đây, các bạn có thể thấy là chúng ta đă sử dụng channel trong ngôn ngữ lập trình Go cho việc xử lý không đồng bộ giữa các sự kiện client và server, các bạn có thể tìm hiểu thêm tại link  http://onsi.github.io/ginkgo/#asynchronous-tests

Sau khi, đã cài đặt xong nội dung unit test chúng ra có thể tiến hành chạy unit test code bằng lệnh sau

```
$ ginkgo -r --randomizeAllSpecs -cover
Running Suite: Gochatapp Suite
==============================
Random Seed: 1459623856 - Will randomize all specs
Will run 1 of 1 specs

2016/04/03 02:04:19 number of connections:  1
2016/04/03 02:04:19 on connection User-lppmL9nxtgT3FaL9oi1k
2016/04/03 02:04:19 joined_message hadv
Test finished!!!
•
Ran 1 of 1 Specs in 0.060 seconds
SUCCESS! -- 1 Passed | 0 Failed | 0 Pending | 0 Skipped PASS
coverage: 41.0% of statements

Ginkgo ran 1 suite in 2.410273465s
Test Suite Passed
Dangs-MacBook-Pro:
```

Giả sử, thay đổi một chút code để cố tình làm cho test case thất bại thì các bạn có thể nhận được kết quả như sau

```
• Failure [0.063 seconds]
Chat
/Users/hadv/work/src/github.com/hadv/gochatapp/chat_test.go:73
  join chat
  /Users/hadv/work/src/github.com/hadv/gochatapp/chat_test.go:72
    join chat sucessfully
    /Users/hadv/work/src/github.com/hadv/gochatapp/chat_test.go:71
      join chat sucessfully [It]
      /Users/hadv/work/src/github.com/hadv/gochatapp/chat_test.go:70

      Expected
          <string>: hadv
      to equal
          <string>: dang

      /Users/hadv/work/src/github.com/hadv/gochatapp/chat_test.go:69
------------------------------


Summarizing 1 Failure:

[Fail] Chat join chat join chat sucessfully [It] join chat sucessfully 
/Users/hadv/work/src/github.com/hadv/gochatapp/chat_test.go:69

Ran 1 of 1 Specs in 0.064 seconds
FAIL! -- 0 Passed | 1 Failed | 0 Pending | 0 Skipped --- FAIL: TestGochatapp (0.06s)
FAIL
coverage: 41.0% of statements

Ginkgo ran 1 suite in 2.144790233s
Test Suite Failed
```

Sau đây là ảnh chụp màn hình `console` sau khi chạy unit test không thành công.

![unit test failed](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/gochatapp_%E2%80%94_-bash_%E2%80%94_176%C3%9753_and_S%E1%BB%AD_d%E1%BB%A5ng_ginkgo_async_%C4%91%E1%BB%83_vi%E1%BA%BFt_unit_test_cho_c%C3%A1c_%E1%BB%A9ng_d%E1%BB%A5ng_socket_io.jpg_5x5mpz0yhi)

Như vậy với sự hỗ trợ mạnh mẽ từ thư viện `Ginkgo` chúng ta hoàn toàn có thể dễ dàng viết được unit test cho các ứng dụng phát triển bằng `socket.io`. Các bạn có thể xem toàn bộ mã nguồn và unit test trên link github của tôi: https://github.com/hadv/gochatapp



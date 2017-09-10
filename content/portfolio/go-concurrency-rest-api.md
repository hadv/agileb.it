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
hẳng là hồi học đại học năm thứ nhất đại học, tôi vẫn tiếp tục hì hục ngồi code các bài toán trên ngôn ngữ lập trình `Pascal`, ngôn ngữ lập trình mà tôi đã được học từ năm lớp 10 ở trường PTTH, thằng bạn cùng lớp thấy thế nên nó bảo tôi: "Nếu mày muốn thành lập trình viên chuyên nghiệp thì kiểu gì mày cũng phải học `C/C++`".
Nên tôi và nó có mò lên hiệu sách trên phố sách đình đám nhất Hà Thành, *Tràng Tiền*, lê la khắp các hiệu sách để tìm mua 1 quyển lập trình `C++` và sau đó có mua thêm một quyển lập trình `Assembly` của *Peter Norton* khá nổi tiếng và phổ biến những năm đó thì phải.
Tôi vẫn nhớ như in là tôi đã rất ấn tượng với bìa sách hình chân dung ở tư thế đang khoanh tay trước ngực của Peter Norton. Một phong thái tự tin và rất thông minh.

Thời của tôi là vậy, các thế hệ Y sau này chắc sẽ không còn học C nữa, mà có thể là `Haskell` hay một ngôn ngữ nào mới mẻ khác mà chúng ta chưa biết.
Các ngôn ngữ lập trình hiện đại thì ngày càng dễ dàng cho lập trình viên hơn. Các lập trình viên ở *Google* cho ra đời `Golang` chắc cũng nhằm mục đích đơn giản hoá việc lập trình trên ngôn ngữ `C` mà vẫn đạt được hiệu năng cao.

Hôm nay, tôi làm một code mẫu tạo REST API đơn giản bằng Golang để thấy được việc lập trình *concurrency* bằng Golang nó trở nên đơn giản đến mức nào nếu so sánh với việc làm tương tự trên ngôn ngữ lập trình C/C++ hồi xưa.
Tôi xin nhấn mạnh là `C/C++` của thời *ông bà anh* nhé vì bản thân `C/C++` vẫn đang phát triển và mới cho ra phiên bản năm 2017 có nhiều điểm mới mà tôi cũng không biết cụ thể nó là cái gì? :)
Và ở một cách tiếp cận khác, tiếp cận hơi khác nhưng gần đây các ngôn ngữ lập trình đều có thư viện `reactive programming` cũng khá thuận tiện cho việc xử lý các yêu cầu asynchronous nhằm tăng độ response của các ứng dụng.
Tuy nhiên, tuỳ vào yêu cầu bài toán và cách thiết kế hệ thống mà chúng ta cần lựa chọn sử dụng cách tiếp cận nào cho phù hợp và dễ dàng hơn.

*Yêu cầu* của bài toán là viết một API trả về nhiệt độ trung bình từ nhiều nguồn số liệu của một thành phố.
Thông thường, nếu sử dụng các lập trình tuần tự thì chúng ra phải lấy nhiệt độ của thành phó của từng nguồn cung cấp rồi tính ra được nhiệt độ trung bình.
Giả dụ chúng ta có 10 nguồn cung cấp số liệu và để lấy nhiệt độ của mỗi nguồn mất 1 giây thì tổng thời gian cho việc lấy nhiệt độ trung bình của API sẽ là 10 giây.
Opsss, 10 giây, một thời gian khó có thể chấp nhận cho một API trả về kết quả.

Giải pháp để tăng hiệu năng cho API ở đây là sử dụng lập trình `go concurrnecy` kết hợp với `channel` để có thể cùng một lúc lấy được nhiệt độ của các nguồn cung cấp trong cùng một thời điểm.
Kết quả là chúng ra chỉ cần, tương đối, 1 giây để có thể lấy được kết quả từ 10 hay nhiều nguồn số liệu khác nhau. Một sự khác biệt đáng kinh ngạc?

Cụ thể các bước tiến hành cài đặt như sau.

* Trước tiên thì chúng ta định nghĩa một interface chung để lấy nhiệt độ của một thành phố từ các nguồn số liệu khác nhau

```
type weatherProvider interface {
	temperature(city string) (float64, error)
}
```

* Vì mỗi nguồn số liệu có cách lấy số liệu khác nhau nên chúng ta cần khai báo các `struct` tương ứng để cài đặt. 
Ở đây để đơn giản thì tôi lấy từ 2 nguồn số liệu là `OpenWeatherMap` và `apixu`


```
type openWeatherMap struct {
	apiKey string
}

func (w openWeatherMap) temperature(city string) (float64, error) {
	resp, err := http.Get("http://api.openweathermap.org/data/2.5/weather?APPID=" + w.apiKey + "&q=" + city)
	if err != nil {
		return 0, err
	}

	defer resp.Body.Close()

	var d struct {
		Main struct {
			Kelvin float64 `json:"temp"`
		} `json:"main"`
	}

	if err := json.NewDecoder(resp.Body).Decode(&d); err != nil {
		return 0, err
	}

	log.Printf("openWeatherMap: %s: %.2f", city, d.Main.Kelvin)
	return d.Main.Kelvin, nil
}

type apixu struct {
	apiKey string
}

func (w apixu) temperature(city string) (float64, error) {
	resp, err := http.Get("http://api.apixu.com/v1/current.json?key=" + w.apiKey + "&q=" + city)
	if err != nil {
		return 0, err
	}

	defer resp.Body.Close()

	var d struct {
		Observation struct {
			Celsius float64 `json:"temp_c"`
		} `json:"current"`
	}

	if err := json.NewDecoder(resp.Body).Decode(&d); err != nil {
		return 0, err
	}

	kelvin := d.Observation.Celsius + 273.15
	log.Printf("apixu: %s: %.2f", city, kelvin)
	return kelvin, nil
}
```

* Sau khi, đã có thể lấy được số liệu từ các nguồn cụ thể thì chúng ta sẽ sử dụng `go concurrency` và `channel` để cài đặt cách thức để lấy số liệu của các nguồn được diễn ra cùng một lúc để giảm thiểu thời gian chờ đợi.

```
func (w multiWeatherProvider) temperature(city string) (float64, error) {
	// Make a channel for temperatures, and a channel for errors.
	// Each provider will push a value into only one.
	temps := make(chan float64, len(w))
	errs := make(chan error, len(w))

	// For each provider, spawn a goroutine with an anonymous function.
	// That function will invoke the temperature method, and forward the response.
	for _, provider := range w {
		go func(p weatherProvider) {
			k, err := p.temperature(city)
			if err != nil {
				errs <- err
				return
			}
			temps <- k
		}(provider)
	}

	sum := 0.0

	// Collect a temperature or an error from each provider.
	for i := 0; i < len(w); i++ {
		select {
		case temp := <-temps:
			sum += temp
		case err := <-errs:
			return 0, err
		}
	}

	// Return the average, same as before.
	return sum / float64(len(w)), nil
}
```

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

```
func main() {
	mw := multiWeatherProvider{
		openWeatherMap{apiKey: "5bd6d7d469feee97788f51744f8c2910"},
		apixu{apiKey: "ff8b321075a54e7288794851162712"},
	}

	http.HandleFunc("/weather/", func(w http.ResponseWriter, r *http.Request) {
		begin := time.Now()
		city := strings.SplitN(r.URL.Path, "/", 3)[2]

		temp, err := mw.temperature(city)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}

		w.Header().Set("Content-Type", "application/json; charset=utf-8")
		json.NewEncoder(w).Encode(map[string]interface{}{
			"city": city,
			"temp": temp,
			"took": time.Since(begin).String(),
		})
	})

	http.ListenAndServe(":8080", nil)
}
```

Như vậy, chúng ta có thể dễ dàng cài đặt một REST API hiệu quả bằng cách sử dụng `go concurrency` và `channel`. 
Source code mẫu các bạn có thể xem trên github: https://github.com/hadv/go-concurrency-rest-api
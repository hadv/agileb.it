---
title: "Sử dụng GVM để quản lý các phiên bản của Golang"
date: 2017-09-03T15:45:42+07:00
draft: true
image: "img/portfolio/gopher.png"
---

Tương tự như [NodeJS](https://nodejs.org), [Golang](https://golang.org) cũng có những công cụ để cài đặt và sử dụng nhiều phiên bản của trình biên dịch trên cùng một máy một cách nhanh chóng và gọn nhẹ. [Moovweb](http://moovweb.com) đã phát triển một công cụ, GVM - Go Version Management, để làm việc này. Để cài đặt [GVM](https://github.com/moovweb/gvm) các bạn chỉ cần chạy lệnh sau trên console

```
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

> Nếu bạn sử dụng `zsh` thì chỉ cần đổi `bash` thành `zsh` là được.

Một chú ý nhỏ là trước khi cài đặt [GVM](https://github.com/moovweb/gvm) thì chúng ta phải cài đặt các thư viện liên quan trước.

## MacOSX
* Cài `Mercurial` từ trang web: http://mercurial.berkwood.com/
* Cài `Xcode Command Line Tools` từ App Store. Thông thường là đã đi kèm khi chúng ta cài đặt `Xcode` rồi.

## Linux
### Debian/Ubuntu
Chạy lệnh `sudo apt-get` bên dưới để cài đặt toàn bộ các chương trình liên quan

```
$ sudo apt-get install curl git mercurial make binutils bison gcc build-essential
```

### Redhat/Centos
Chạy các lệnh sau để cài đặt các chương trình liên quan

```
$ sudo yum install curl
$ sudo yum install git
$ sudo yum install make
$ sudo yum install bison
$ sudo yum install gcc
$ sudo yum install glibc-devel
```

* Ngoài ra cần cài đặt thêm `Mercurial ` từ trang web http://pkgs.repoforge.org/mercurial/

Sau khi cài đặt thành công GVM, chúng ta có thể sử dụng GVM để cài đặt các phiên bản của Go một cách đơn giản.

> Từ phiên bản `Go 1.5` trở lên thì Google đã loại bỏ việc sử dụng trình biên dịch C khỏi toolchain và thay thế bằng một chương trình được viết chính bằng Go nên bắt buộc chúng ta phải cài đặt phiên bản Go 1.4 trước khi cài các phiên bản cao hơn.

Để cài đặt một phiên bản Go nào đó, chúng ta chỉ cần chạy lệnh đơn giản sau đây và sử dụng `go use` để chỉ định phiên bản Go sẽ được sử dụng.

```
$ gvm install go1.4
$ gvm use go1.4 --default
```

Ví dụ để cài đặt và sử dụng Go 1.5 chúng ta chạy các lệnh sau đây. Như đã chú ý ở trên chúng ta phải cài đặt go1.4 trước khi cài đặt các phiên bản cáo hơn.

```
$ gvm install go1.4
$ gvm use go1.4
$ gvm install go1.5
```

Để liệt kê các phiên bản Go đang được cài đặt chúng ta chạy lệnh `gvm list`, phiên bản Go đang được sử dụng thì được đánh dấu bằng ký tự `=>` ở đầu.

```
gvm gos (installed)

   go1.4
=> go1.5
   go1.6
```

> Để liệt kê toàn bộ phiên bản của Go có thể cài đặt thì chúng ta chạy lệnh `gvm listall`


Như vậy, sử dụng GVM chúng ta có thể dễ dàng càt đặt và sử dụng phiên bản bất kỳ của Go một cách nhanh chóng chỉ trong một nốt nhạc :smile:

Các bạn tìm hiểu thêm cách sự dụng gvm bằng cách chạy lệnh `gvm help` nhé! :tongue:


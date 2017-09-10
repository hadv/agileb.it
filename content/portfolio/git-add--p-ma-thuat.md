---
title: "`git add -p` ma thuật"
date: 2017-09-04T15:45:42+07:00
draft: false
image: "img/portfolio/git-add--p.png"
---

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/git-add-p.jpg_ty38z9grkz)

Sử dụng git cho công việc hàng ngày, thông thường chúng ta sử dụng lệnh `git add .` hoặc `git add --all` để thêm toàn bộ những nội dung thay đổi vào trạng thái staging chuẩn bị cho lần commit code tiếp theo. Hoặc nếu chỉ muốn commit một vài file cụ thể nào đó thì chúng ta sẽ add từng file riêng lẻ bằng cách chỉ định đường dẫn đến các file đó như sau

```
$ git add main.go conf/config.yaml
```

Tuy nhiên, trong trường hợp nào đó bạn chỉ muốn commit một phần thay đổi của một file thì có cách nào không? Đây chính là lúc sự ma thuật của `git add -p` thể hiện sức mạnh của mình. Lệnh `git add -p` cho phép chúng ta duyệt qua lần lượt các nội dung thay đổi của từng file mà git gọi mỗi phần thay đổi này là một `hunk`; qua giao diện tương tác trên console chúng ta có quyền được lựa chọn có thực hiện staging từng phần thay đổi cụ thể nào đó hay không bằng cách nhập các ký tự tương ứng. Dưới đây là một số lệnh tương tác phổ biến của `git add -p`. 

> Để xem chi tiết các bạn có thể sử dụng `?` để in ra hướng dẫn như sau.

```
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
```

Có lẽ không nhiều trường hợp chúng ta cần sử dụng đến `git add -p` vì thông thường mỗi lần commit chúng ta sẽ chỉ thay đổi những file cần thiết và dùng `git add --all`. Tuy nhiên, đó đôi khi là một cách làm ẩu nếu chúng ta không quản lý thay đổi một cách cẩn thận. Việc sử dụng `git add -p` giúp chúng ta xem lại và chắc chắn được từng nội dung thay đổi mà chúng ta muốn chứ không commit nhầm những thay đổi ngoài ý muốn hoặc những file không cần thiết.

> Tương tự với `git add -p`, chúng ta có thể sử dụng `git checkout -p` để có thể loại bỏ từng phần thay đổi của một file khỏi thư mục làm việc.


SourceTree của Atlassian cũng có cung cấp đầy đủ các tính năng này trên GUI

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/source_tree.jpg_cmwc45z9b1)


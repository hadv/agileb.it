---
title: "Viết lại lịch sử của git bằng cách sử dụng interactive rebase"
date: 2017-09-04T21:45:42+07:00
draft: false
image: "img/portfolio/git-rebase.png"
---

Trong một vài tình huống do sơ xuất chúng ta có thể commit thiếu file lên git server hoặc ghi nội dung comment chưa được như ý muốn hoặc nội dung comment chưa đúng theo quy tắc chung đã đưa ra. Trong tình huống đó, để sửa nội dung đã commit chúng ra có thể  sử dụng lệnh `git rebase -i` như sau:

> Trước khi sử dụng `git rebase` thì working directory phải đang sạch sẽ. Nếu như bạn đang làm dở công việc mà muốn rebase thì có thể sử dụng `git stash` để lưu tạm công việc còn dang dở vào stash list. Sau đó, có thể lấy ra để tiếp tục công việc.

1. Chạy lệnh`git rebase -i` trên chính branch cần sửa nhưng lùi 1 version bằng cách chỉ định `HEAD~1`

	```
	$ git rebase -i HEAD~1
	```
2. git sẽ hiển thị ra file như sau để cho phép chúng ra sửa đổi nội dung đã commit theo nhu cầu

	```
	pick eb9b5de Add test case for Not Found 404

    # Rebase 9c7458a..eb9b5de onto 9c7458a
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out
 	```

	ở đây ví dụ chúng ra cần sử nội dung comment, thì chúng ra sẽ sửa nội dung file trên thành nưh sau

	```
	e eb9b5de Add test case for Not Found 404
	```

3. Lưu nội dung file trên và thoát khỏi trình soạn thảo. Sau đó, chạy lệnh sau đây để sửa nội dung comment
	```
	git commit --amend 
    ```
 
4. Git sẽ hiển thị trình soạn thảo để cho phép chúng ra sửa nội dung comment như sau
	```
    Add test case for Not Found 404

    # Please enter the commit message for your changes. Lines starting
    # with '#' will be ignored, and an empty message aborts the commit.
    #
    # Date:      Sun Mar 6 11:47:10 2016 +0700
    #
    # rebase in progress; onto 9c7458a
    # You are currently editing a commit while rebasing branch 'master' on '9c7458a'.
    #
    # Changes to be committed:
    #       modified:   api/server_test.go
    #    
    ```
 
5. Giả dụ chúng ta cần sửa comment code đẻ thêm số issue `#112` vào comment như sau
	```
    Add test case for Not Found 404 #112
	```

6. Gõ lệnh `git rebase --continue` để tiếp tục và hoàn thành `rebase`. Nếu trong quá trình `rebase`, bạn suy nghĩ lại không muốn thực hiện `rebase` nữa thì có thể sử dụng lệnh `git rebase --abort`

7. Đến bước này chúng ra đã hoàn thành việc thay đổi nội dung comment của lịch sử commit. Bước cuối cùng là push thay đổi lên git server là xong.
	```
    $ git push --force origin master
    ```
 
 	Một chú ý nhỏ ở đây, chúng ta phải sử dụng `push --force` vì local và remote repo đang bị lệch về version, `not-fast-forward`
 
	> Hãy cẩn thận khi sử dụng `push --force` vì có thể bạn sẽ làm mất source code của người khác.


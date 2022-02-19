---
layout: post
title: "Entry point"
date: 2022-02-19 08:00:00 +0000
categories: code
---

Cho dù bạn viết bất cứ ngôn ngữ nào, luôn có một điểm khởi đầu cho chương
trình. Mỗi điểm khởi đầu có cái hay của riêng nó. Bài viết này sẽ so sánh
các 2 khởi đầu phổ biến của các ngôn ngữ.

### Hàm `main()`

##### C++
```cpp
int main() {

}
```

##### Java
```java
class Main {
    public static void main(String[] args) {
    }
}
```

##### Haskell
```haskell
main = return 0
```

##### Pascal
```pascal
begin

end.
```

Đây có lẽ là điểm khởi đầu phổ biến nhất, có lẽ do bắt nguồn từ ngôn ngữ C
và nhiều ngôn ngữ sau này kế thừa từ C. Điểm đặc trừng là cho dù mã nguồn có
viết gì đi chăng nữa, code được biên dịch sẽ bắt đầu chạy từ hàm `main()` này
(tất nhiên ngoài trừ các biến global, nhưng chúng ta sẽ không nói nó ở đây.)

Có một ưu điểm là hàm `main()` này khi chạy có thể tương tác một cách trực quan
với shell. Phần tham số (argument, phần ở trong `()`) có thể sửa lại để nhận vào
tham số khi gọi ở terminal, và thường giá trị trả về là exit code mà shell nhận
được. Chẳng hạn, trong C++, ta có thể viết như sau:

```cpp
int main(int argc, char* argv[]) {
    return 0;
}
```

Hàm trên sẽ nhận vào một danh sách các tham số `argv`, độ dài `argc`, kiểu của
mỗi tham số là `char *` (hay được biết đến là _null-terminated string_). Exit
code của chương trình sau khi chạy xong sẽ là `0`. Bạn có thể đọc thêm ở đây:
[C++ - Main function](https://en.cppreference.com/w/cpp/language/main_function)

Tùy vào đặc điểm ngôn ngữ mà cách thể hiện `main()` được thay đổi. Chẳng hạn,
một số ngôn ngữ nặng về hướng đối tượng như Java hay C# sẽ yêu cầu bạn cho vào
bên trong một class, và `main()` lúc đó sẽ là một phương thức (method). Hay
trong Pascal, nó sẽ chỉ đơn giản là `begin` và `end.`


### Từ dòng đầu tiên (không có `main()`)

##### JavaScript
```js
console.log("No main()")
```

##### Python
```py
print("No main()")
```

Sau này, sự xuất hiện phổ biến của trình thông dịch (intepreter) đã đem lại
kiểu khởi đầu mới: loại bỏ hàm `main()`. Mọi thứ được đọc từ trên xuống dưới,
từ đầu đến cuối file. Theo logic thì rõ ràng `main()` là thừa thãi, vì khi
bạn mở trình thông dịch, bạn có thể nhập vào chương trình và nhận lại kết quả
tức thì luôn mà không cần đợi biên dịch thành mã máy. Tuy nhiên, điểm bất lợi
là khi bạn chia theo module, làm thế nào để bạn xác định được đâu là điểm bắt
đầu? Ngôn ngữ như Python có một giải pháp: kiểm tra xem có phải điểm khởi đầu
không:

```py
if __name__ == "__main__":
    pass
```

Sẽ có một số ngôn ngữ trộn lẫn cả hai. Compiler GHC (Glasgow Haskell Compiler)
khi ở chế độ intepreter sẽ không có hàm `main()`, nhưng khi biên dịch sẽ yêu
cầu bạn phải có.

Từ góc nhìn hiện tại, có khi `main()` chỉ là di sản của quá khứ. Trong một số
trường hợp, `main()` lại là yếu tố quan trọng trong phân bố code theo module.
Nhưng sau cùng, việc có `main()` hay không hoàn toàn dựa vào quyết định của
người thiết kế ngôn ngữ. Vậy nên viết code thế nào bạn cũng nên bám sát vào
thiết kế hay tiêu chuẩn của ngôn ngữ đó.
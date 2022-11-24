---
layout: post
title: "Language Review #1: JavaScript"
date: 2022-10-31 05:00:00 +0000
categories: code
tags: language
---

Chào mừng bạn đến với sêri mới của blog: Language Review. Sêri này sẽ đi sâu
vào các ngôn ngữ lập trình, đánh giá khả năng ứng dụng cũng như mức độ dễ dàng
hay khó khăn khi sử dụng. Phần lớn thông tin sẽ là trải nghiệm của bản thân tôi.
Mỗi bài viết cũng sẽ là tổng hợp các tài liệu đương thời về ngôn ngữ.
Bạn đọc có thể tham khảo khi muốn sử dụng ngôn ngữ.

Số đầu tiên, chúng ta sẽ nói về ngôn ngữ phổ biến trên nền Web hiện nay: JavaScript.

## Tài liệu

- Đặc tả ECMAScript:
[ECMA-262](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/),
[ECMA-402](https://www.ecma-international.org/publications-and-standards/standards/ecma-402/)
- Đặc tả [HTML Living Standard](https://html.spec.whatwg.org/multipage/)
- Tài liệu (tiếng Anh): [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

## Lược sử

JavaScript, hay viết tắt là JS, là một trong những công nghệ trụ cột của thế
giới Web. Khi Web mới sinh ra, nguyên lý ban đầu chỉ là chia sẻ những văn bản
thuần túy. Tính chất của Web lúc bấy giờ hoàn toàn tĩnh: những đoạn văn là đối
tượng chính, chuyển trang là cốt yếu. JavaScript làm thay đổi cục diện bằng
việc đem lại tính _động_ cho Web lúc bấy giờ. Bạn có thể chơi game, nhắn tin
trên Web, và được nhận cập nhật thông tin gần như tức thời.

JavaScript được tạo ra trong thời gian ngắn ngủi bởi các kĩ sư ở Netscape với
mong muốn nó giống Java và có thể tốc kí (script) được. Để xử lý nhanh chóng,
JavaScript nay trở thành ngôn ngữ JIT (biên dịch khi chạy). Tuy nhiên, không
tránh khỏi việc nhiều tính năng JavaScript thừa hưởng những sai sót từ các ngôn ngữ khác.

Phiên bản trong bài đánh giá là ECMAScript 2022. ECMAScript là tiêu chuẩn chung
cho các phiên bản JavaScript được triển khai bởi các phần mềm khác nhau. Nó cũng
hay được viết tắt là ES (chẳng hạn, ES 2022).

## Tôi có thể chạy bằng cách nào?

- Lựa chọn số 1 là những trình duyệt Web bạn đang sử dụng: Chrome, Firefox, ...
Tất cả đều có JavaScript engine.
- Lựa chọn số 2 là tải và dùng những runtime độc lập như: Node.js, Deno, bun.
Tương tự như trên, nhưng chúng ta sẽ có thêm một vài tính năng bên ngoài tương
tự các ngôn ngữ khác như nhập xuất với console, tương tác với file.
- Compile JavaScript ra mã máy khả thi, nhưng không thể đánh bại được engine như V8.
Giải pháp thịnh hành hơn là ghép executable của Node.js và code JavaScript vào
chung một binary (Tìm nexe).

## Tính năng nổi bật

### Phá (kiểu) cách, nhưng tốc độ cao

Hệ thống kiểu trong JavaScript là động, không gắn liền kiểu nhất định với một
biến. Biến `a` nay là string, mai có thể là number, chẳng có ai cản.
```js 
let a = "Welcome";
a = 2;              // OK
a = "JavaScript";   // OK
```
Ấy vậy mà chớ xem thường, JavaScript đôi khi có thể ngang bằng với C++, một ngôn
ngữ bậc thấp, gần với phần cứng hơn (trong
điều kiện code C++ không được tối ưu bằng tay). Để có thể trở thành một nền
tảng của Web hiện đại, JavaScript ắt cần phải nhanh, nhưng điều kì diệu gì
đã giúp cho nó nhanh? Câu trả lời nằm ở Engine: phổ biến là V8 và SpiderMonkey.
Các engine đều gồm Garbage Collector và Just-in-Time Compiler rất hiệu quả trong việc
đưa ra dự đoán và khởi động (warmup). Khi tiếp nhận, bản thân mã nguồn JavaScript cũng được
biên dịch trước (Ahead-of-Time compiling) và engine sẽ viết lại dưới dạng tối ưu hơn.

Một ví dụ có thể kể đến: V8 ngầm chia mảng thành hơn 20 kiểu khác nhau. Mỗi
kiểu sẽ có cách tối ưu khác nhau. Nếu đoán là mảng các số liên tiếp
(`[1, 10, 4]` chẳng hạn), V8 sẽ bật SIMD (đơn-lệnh-đa-dữ liệu) cho mảng và chúng
ta được hưởng tốc độ của tính toán đa chiều (vector) nhanh hơn nhiều lần bình
thường. Xem thêm: [Elements kinds in V8](https://v8.dev/blog/elements-kinds)

Tất cá tối ưu đều nhằm cho chương trình nhanh hơn, nhưng không đảm bảo về lượng
bộ nhớ mà engine sử dụng.

### Đánh giá kiểu - vừa tiện vừa dở

Trong JavaScript có hai loại kiểu chính, không hiện diện trực tiếp ở mặt code:
- Loại thứ nhất là **nguyên thủy** (primitive),
bao gồm các kiểu như `number`, `string`, `boolean`. Loại này có đặc điểm dễ
copy (trivially copyable), nên khi gọi câu lệnh như `console.log(100)` thì
giá trị `100` sẽ được copy truyền vào trong hàm.
- Loại thứ hai là **đối tượng** (object). Loại này thường gắn liền với mô hình
OOP, có trạng thái riêng (state) và hàm dành riêng (method) cồng kềnh. Chính vì
đó, loại này không được copy mà chỉ được truyền bằng đường dẫn (reference),
rất giống Java. Ví dụ như sau:
```js
const person_a = { name: "Bob" };
const person_b = person_a;      // a và b cùng trỏ đến một object
person_b.name = "Alice";
console.log(person_a.name);     // Alice
```

Quy luật ngầm trên dở ở chỗ, nếu bạn muốn tạo một mảng đa chiều trong JavaScript:
```js
const matrix_1 = Array(10).fill(Array(10).fill(0));
const matrix_2 = Array(10).fill(0).map(() => Array(10).fill(0));
```
Thứ trả về ở `matrix_1` thực chất là một mảng 10 đường dẫn, và tất cả đều trỏ tới
cùng một mảng trong bộ nhớ do `Array` thuộc loại đối tượng, `.fill` sẽ copy đường
dẫn chứ không copy giá trị. Thứ chúng ta cần nằm ở `matrix_2`: một mảng 10 đường
dẫn, mỗi đường dẫn trỏ đến mảng khác nhau. Để có thể tạo ra và lưu lại đường dẫn
đến đối tượng khác nhau như vậy, chúng ta sử dụng hàm kết hợp với hàm bậc cao
(Higher-order Function) `.map`.

Cũng nói luôn, mô hình quản lý bộ nhớ của JavaScript là sử dụng Garbage Collection:
bộ nhớ còn sử dụng thì giữ, hết dùng rồi thì vứt đi, mở
chỗ cho lưu trữ thông tin mới.

Tuy vậy, phần lớn các tính năng khác không hề giống Java. Nếu có ai hỏi Java và
JavaScript có giống nhau hay liên quan đến nhau không, câu trả lời vẫn là không.

### Không có lỗi trước mắt, chỉ có... chương trình không chạy

Đối tượng chính của Web là đại đa số người thường, không phải lập trình viên.
Việc xuất hiện trước mắt màn hình đỏ kèm dòng chữ như văn tự ngoài hành tinh
chẳng thể nào giúp gì cho trải nghiệm người dùng. Vì đó, JavaScript chọn sự
im lặng: Dù có lỗi xảy ra bên trong nhưng bề ngoài vẫn như thường, không gì khác
biệt ngoài việc chương trình chạy không đúng như mong đợi của bạn!

Mượn ví dụ ban đầu, những câu lệnh sau hoàn toàn hợp lệ:

```js 
let a = "Welcome";
a = 2;              // OK, giá trị là 2
a += "JavaScript";  // OK, giá trị là "2JavaScript"
a -= 10;            // OK, giá trị là NaN
```

Ngoài ra, hãy xét đoạn chương trình sau:
```js
function print(a) {
  console.log(a);
}
```
Nếu bạn gọi `print(2)` hay `print("Hi, John!")` thì mọi việc hoàn toàn êm xuôi.
Đến cả in một mảng `print([1, 2, 3])` cũng chẳng sao cả. Thế còn
`print([1, 2, 3] + [4, 5])`, bạn nghĩ chương trình sẽ in ra gì?
```
> print([1, 2, 3] + [4, 5])
1,2,34,5
```
Số 34 ở đâu ra? Để hiểu được điều gì đã diễn ra, bạn cần phải nắm rõ nguyên lý.
Khi thực hiện các phép tính đối với `+` trong JavaScript, đối với
kiểu khác nhau thì sẽ đưa về chung một kiểu, mà ưu tiên số một là kiểu string.
`[1, 2, 3]` khi đổi ra string sẽ là `1, 2, 3`; khi chúng ta `+` hai string, kết
quả sẽ là một string mới ghép nối tiếp bởi hai. Đến đây thì bạn hết bất ngờ nhỉ?

Vẫn ví dụ đó, nếu câu lệnh gọi lúc này là `print()` thì sao? Ngữ pháp không hợp lệ?
Chuyện đó không dễ dàng đến thế với JavaScript.
Nếu thừa thì JavaScript sẽ cắt bớt, mà nếu thiếu thì JavaScript sẽ bù đắp bằng
`undefined`. Thật vậy:
```
> print(1, "Hello")
1
> print()
undefined
```

Lời khuyên hữu hiệu nhất để đối phó với sự _tùy hứng_ này là sử dụng
linter như ESLint đế phân tích tĩnh chương trình và tìm ra lỗi có thể phát sinh.
Hoặc ít nhất, đừng nhìn mà đoán, hãy tham khảo kĩ
tài liệu trên MDN hay ECMA-262 để nắm rõ các _bất quy tắc_ như trên.

Xem thêm: [denysdovhan/wtfjs - 🤪 A list of funny and tricky JavaScript examples](https://github.com/denysdovhan/wtfjs)

### Event-loop để chạy không đồng bộ

JavaScript dùng cho Web mà dừng lại chờ mỗi yêu cầu như gửi file thì sẽ rất chậm,
tất cả máy chủ dùng Node.js sẽ bị DoS (Từ chối dịch vụ). JavaScript thuần tiêu
chuẩn sử dụng vòng lặp sự kiện (Event-loop) để hỗ trợ lập trình không đồng bộ.

Khi chạy chương trình, JavaScript giữ một ngăn xếp ngầm chạy bên lề. Khi một
lệnh không đồng bộ được đưa ra từ chương trình chính, JavaScript sẽ đẩy nó vào
ngăn xếp và chạy nó cho đến khi kết quả trả về. Khi ấy, chương trình chính vẫn
diễn ra như thường. Kể cả chương trình chính chạy xong rồi, event-loop vẫn sẽ
đợi cho đến khi ngăn xếp đẩy hết các lệnh không đồng bộ.

Xem thêm:
[What the heck is the event loop anyway? | Philip Roberts | JSConf EU](https://youtu.be/8aGhZQkoFbQ)

Nhân tiện, một tác vụ phụ chạy không đồng bộ trong JavaScript có thể được biểu
diễn bằng một `Promise`. Bạn có thể dễ dàng biến một hàm thành chạy không đồng bộ
bằng cách thêm `async` vào trước. Để có thể đợi một `Promise`, đơn giản dùng `await`
(trong một hàm `async`).

Trong đặc tả HTML Living Standard có bổ sung Web Workers, cho phép JavaScript
phân phối và chạy đa luồng thông qua các Worker. Tính năng này hỗ trợ ở Chrome,
Firefox, hay hỗ trợ một phần ở Node.js. Đây là một trong những ví dụ của tính năng
không nằm trong tiêu chuẩn ECMAScript, nhưng đa số runtime hỗ trợ.

Xem tiếp [_Phần 2_]({% post_url 2022-11-24-langreview-2 %})
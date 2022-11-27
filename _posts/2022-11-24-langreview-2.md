---
layout: post
title: "Language Review #2: JavaScript (p2)"
date: 2022-11-24 16:30:00 +0000
categories: code
tags: language
---

Số thứ 2 của sê-ri, chúng ta sẽ bàn nốt những đặc điểm còn lại của JavaScript.
Nếu bạn chưa đọc phần trước, hãy xem:
[Language Review #1: JavaScript](/-blog{% post_url 2022-10-31-langreview-1 %}).

## Mẫu hình lập trình (Paradigms)

Do JavaScript nằm trong gia đình ngôn ngữ C, mẫu hình lập trình có trong
JavaScript cũng thừa hường từ các ngôn ngữ trong nhóm này, chẳng hạn: lập trình
hương đối tượng (OOP), lập trình hàm.

### Lập trình thủ tục (Procedural)

`function` trong JavaScript có thể `return` tùy ý mà không phải thống nhất kiểu.
Bạn hoàn toàn có thể trả về 1 hay "abc" trong cùng một hàm. Nếu không trả về gì,
thì kết quả khi gán giá trị của hàm sẽ là `undefined`. Lưu ý, hàm trong định
nghĩa này có thể hoisting, tức được xử lý trước tiên khi bước vào một phạm
vi. Chẳng hạn,

```js
console.log(plus1(99));

function plus1(n) {
    return n + 1;
}
```
sẽ chạy hoàn toàn bình thường và in ra 100.

### Lập trình hàm (Functional)

Trong JavaScript, hàm có thể được định nghĩa như là một biểu thức và coi như là
một đối tượng. Có những ba kiểu định nghĩa:
1. `function name() {}`
2. `function () {}`
3. `() => {}`

Ba cách viết này không giống nhau về việc sử dụng. Số 1 giống với hàm bình thường
như đã nói ở trên, nhưng khác ở chỗ có thể dùng như biểu thức và không được hoisting.
Số 2 giống số 1, nhưng bỏ đi yêu cầu sử dụng tên cho hàm. Số 3 khác số 1 và 2
về phạm vi của `this`. Nguyên do mỗi khi tạo môi trường mới, chúng ta sẽ có
`this` riêng cho mỗi hàm. Nếu bạn viết code hướng đối tượng, bạn sẽ cảm thấy khó
khăn trong tiếp cận chính Đối tượng. Chẳng hạn, ví dụ từ MDN:
```js
function Person() {
    this.age = 0;

    // Sau mỗi một giây, tăng một tuổi
    setInterval(function growUp() {
        thís.age++;
    }, 1000);
}
```
`this` ở `this.age++` sẽ được hiểu là của hàm `growUp`. Do hàm `growUp` không
định nghĩa thuộc tính `age` trước đó, nó sẽ lấy là `undefined`, cộng với 1 ra
NaN, mà không động gì đến `this.age` nằm ở Person. Số 3 được thiết kế để khắc
phục vấn đề này bằng cách thừa hưởng `this` của phạm vi trên nó.

Đọc thêm tại 
[MDN - Hàm](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)

### Lập trình hướng đối tượng (Object-oriented)

Lí giải cho việc tại sao có cách thức số 3, OOP trong JavaScript bản chất là
`function`, với các biến được gán vào `this` của hàm và các phương thức được cho
vào trong một thuộc tính gọi là `prototype`.

Thật vậy, khi bạn viết class:
```js
class Animal {
    constructor() {
        console.log("Tạo class Animal");
    }
    walk() {}
}
```

thì nó sẽ gần giống (về mặt tính năng) với:
```js
function Animal() {
    function _constructor() {
        console.log("Tạo class Animal");
    }
    function walk() {}
    this.prototype.walk = walk;

    _constructor();
};
```

### Lập trình hướng sự kiện (Event-driven)

Do chủ chốt của JavaScript là làm cho văn bản HTML trở nên động, JavaScript
sẽ không nhất thiết phải bắt đầu từ `main` mà thay vào đó gắn vào các element
của văn bản và đợi sự kiện (event) tác động.
Event-loop nói trong phần trước cũng có thể coi là một phần của lập trình hướng
sự kiện, khi luồng điều khiển (control-flow) được đan xen bởi các sự kiện và hàm
chạy không đồng bộ.

### Lập trình reflection

JavaScript có thể gọi và chạy gián tiếp các phương thức trong lớp thông qua
`Reflect`.
JavaScript cũng có `eval` cho phép chạy JavaScript dưới dang xâu,
mà vẫn thừa hưởng các yếu tố ở môi trường chạy bao quanh nó. Ví dụ từ
[Wikipedia](https://en.wikipedia.org/wiki/Reflective_programming#JavaScript):
```js
// Không dùng reflection
const foo = new Foo();
foo.hello();

// Dùng reflection
const foo = Reflect.construct(Foo);
const hello = Reflect.get(foo, 'hello');
Reflect.apply(hello, foo, []);

// Dùng eval
eval('new Foo().hello()');
```

## Hệ sinh thái

### Mô-đun

JavaScript được thiết kế là ngôn ngữ script cho Web (Web scripting language).
Đặc tả đầu tiên của JavaScript (ECMAScript 1st edition) có mô tả các Đối tượng
có sẵn (Built-in Object) cung cấp các hàm, tương đương như các thư viện có sẵn
ở các ngôn ngữ được biên dịch. Từ góc độ khác, JavaScript ban đầu chỉ có 2 phạm
vi môi trường: hàm và global.

Khi ứng dụng viết bằng JavaScript xuất hiện ngày một nhiều, tần suất tái sử dụng
code tăng lên, dẫn đến sự xuất hiện của vấn đề mô-đun hóa code. Từ đó đến nay đã
có nhiều giải pháp; ưa chuộng và phổ biến hơn cả bao gồm: CommonJS và AMD
(Asynchronous Module Definition). CommonJS được dùng nhiều ở Node.js, nhưng lại
không tương thích với môi trường trình duyệt. Thay vào đó, lập trình viên ở
frontend hay sử dụng AMD để có thể viết ừng dụng web. Sau cùng, ECMAScript bản
thứ 6 (ECMAScript 6th edition hay ES6) đã cho ra ESModule, nay được chấp nhận
ở tất cả các môi trường.

Tuy nhiên, vì tính tương thích ngược (backward compatibility), và nhiều chương
trình được viết trước đó vẫn đang chạy và bảo trì, việc sử dụng hệ thống mô-đun
mới vẫn bị phân mảnh. May thay, chúng ta có sự xuất hiện của transpiler, một 
chương trình giống trình biên tập, nhưng thay vào đó, kết xuất sẽ là ngôn ngữ
lập trình thay vì mã máy hay bytecode). JS-JS transpiler cho phép tạo ra mã
tương thích với phiên bản cũ từ phiên bản mới hơn của ngôn ngữ. Phổ biến hiện
tại đang có Babel, SWC, esbuild.

Node.js còn có hỗ trợ cho mô-đun gốc (native module, được biên tập trên OS gốc),
cho phép JavaScript gọi hàm viết bằng C, C++ hay ngôn ngữ thấp tưong tự. 

### Thư viện - nhiều là tốt?

Hệ sinh thái JavaScript thay đổi liên tục, thư viện và khung phần mềm (framework)
có nhiều vô kể. Điều này giúp cho lập trình viên không phải viết lại và code
thường xuyên được quét lỗ hổng bảo mật. Lạm dụng điều này đã dẫn đến sự xuất
hiện của vô số các thư viện "mini" như: _is-even_, _is-odd_. _is-even_ thì
phụ thuộc vào _is-odd_. Polyfill (code thay thế khi hàm/thư viện không có sẵn
trong môi trường) thì tràn lan. Mô-đun khi cũ thì nhiều lúc không được phát
triển nữa, có lỗi cũng cam chịu hoặc tự sửa tay. Thật chẵng ra làm sao.

Vậy nên, bài viết này sẽ không đưa lời khuyên cho việc học một thư viện cụ thể
nào; sau cùng nó cũng sẽ cũ, sẽ không được sử dụng lâu. Thay vào đó, bạn đọc
nên nắm rõ JavaScript và các Design Pattern để có thể nhanh chóng tiếp cận khi
có thư viện mới xuất hiện.

Đọc thêm:
- [Require.js - Web Module](https://requirejs.org/docs/why.html)
- [StackOverflow Khảo sát nhà phát triển 2022 - Công nghệ và khung phần mềm Web phổ biến nhất](https://survey.stackoverflow.co/2022/#section-most-popular-technologies-web-frameworks-and-technologies)

## Khả năng sử dụng

### Thao tác

Quy tắc của JavaScript khó học vì đặc điểm không bộc lộ lỗi. Đa số các lỗi sẽ
chỉ có lỗi khi chạy (runtime error) và lỗi cú pháp (syntax error). Trình phiên
dịch sẽ dừng lại ngay khi gặp lỗi, nhưng thông tin báo lỗi không thực sự giúp
ích trong phần lớn trường hợp. Việc viết JavaScript nên đi kèm với chạy trình
gỡ lỗi (debugger).

Vì là ngôn ngữ cho Web, JavaScript nguyên gốc có thư viện hạn chế khi so với các ngôn ngữ khác.
Chẳng hạn, bạn đọc học qua các ngôn ngữ khác như C có thể cảm thấy không quen khi
JavaScript chỉ có `console` hay `document` và một vài hàm lẻ để nhập xuất kết quả.
Chỉ có môi trường như Node.js có cung cấp các API tương tự các ngôn ngữ khác để
tương tác với Standard Input và Standard Output.

### Ứng dụng

Nếu chương trình hay tác vụ bạn cần chỉ gói gọn dưới 50 dòng lệnh thì JavaScript
rất phù hợp. Với tiềm năng của V8 đã nói ở bài trước, tốc độ của JavaScript sẽ
không kém cạnh các ngôn ngữ khác.

JavaScript hiện có mặt ở khắp mọi nơi: Web, Desktop, nhúng (embedded),
điện thoại (mobile), nên khi đã viết ở một nền tảng, bạn có thể dễ dàng port
sang các nền tảng khác với nỗ lực gần như tối thiểu. Chẳng hạn, ứng dụng desktop
có Electron; ứng dụng điện thoại có React Native, Cordova, hay Ionic; nhúng có
Elk. Thời gian phát triển chương trình ở mọi kích thước đều là rất nhanh đối với
JavaScript. Đặc điểm này tương đồng với đà phát triển của start-up.

Về lâu dài, khó có thể coi JavaScript là một ngôn ngữ ổn định. Phần đa các ứng
dụng sử dụng hệ sinh thái khi được viết sẽ gắn liền với một khung phần mềm nhất
định. Sau đó, chúng ta sẽ chỉ có phát triển tiếp từ cơ sở code có sẵn, nâng cấp
các mô-đun phụ thuộc khi có lỗ hổng bảo mật. Điều đáng lo ngại là khi việc cũ
nhanh có thể ảnh hưởng đến khả năng vận hành của ứng dụng, thì việc chuyển đổi
sang thư viện hay khung phần mềm khác mới hơn sẽ thường xuyên bắt buộc, và điều
đó có thể gia tăng thời gian, công sức, và chi phí trong dự án.

## Tổng kết

- JavaScript về điểm gì cũng nhanh nếu được tận dụng đúng cách.
JavaScript rất dễ viết, nhưng cũng rất dễ sai.
- Không dễ để nắm bắt bản chất JavaScript, nhưng khi đã thành thục thì người
viết có thể dễ dàng tiếp cận công nghệ mới trong hệ sinh thái.
- JavaScript thay đổi nhanh như Web, cần có sự đầu tư để thich nghi.
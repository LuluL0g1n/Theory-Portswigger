# 1. Before continue reading
## JavaScript prototypes and inheritance
JavaScript sử dụng mô hình kế thừa nguyên mẫu (prototypal inheritance model), mô hình này khá khác so với mô hình dựa trên lớp (class) được sử dụng bởi nhiều ngôn ngữ khác.
### What is an object in JavaScript?
Một đối tượng (object) JavaScript về cơ bản chỉ là một tập hợp các cặp key:value được gọi là "thuộc tính" (properties). Tức là 1 Object JS bao gồm các property, mà property là 1 cặp key:value.

``` 
const user =  {
    username: "wiener",
    userId: 01234,
    isAdmin: false
}
```

Có thể truy cập các property của một Object bằng cách sử dụng ký hiệu dấu chấm hoặc ký hiệu dấu ngoặc để chỉ các key tương ứng của chúng.
```
user.username     // "wiener"
user['userId']    // 01234
```

Các property cũng có thể chứa các execute function. Lúc này function được gọi là **method**
```
const user =  {
    exampleMethod: function(){
        // do something
    }
}
```

### What is a prototype in JavaScript?
Mọi Object trong JavaScript được liên kết với một Object khác thuộc loại nào đó, được gọi là prototype của nó.

Theo mặc định, JavaScript sẽ tự động gán cho new Object một trong các prototype tích hợp sẵn của nó.
```
let myObject = {};
Object.getPrototypeOf(myObject);    // Object.prototype

let myString = "";
Object.getPrototypeOf(myString);    // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray);	    // Array.prototype

let myNumber = 1;
Object.getPrototypeOf(myNumber);    // Number.prototype
```
Các Object tự động kế thừa tất cả các property của prototype được chỉ định của chúng, trừ khi chúng đã có property riêng với cùng một khóa key (ví dụ tại Object prototype có 1 property như sau: "lulu":"KMA", nhưng tại chính Object đó cũng có 1 property với key là lulu: "lulu":"KCSC" thì property này không cần được kế thừa, tức là value vẫn là KCSC). Điều này cho phép các nhà phát triển tạo các neww Object có thể sử dụng lại các property và method của các existing Object.

### How does object inheritance work in JavaScript?
Bất cứ khi nào bạn tham chiếu một property của một Object, trước tiên, công cụ JavaScript sẽ cố gắng truy cập property này trực tiếp trên chính Object đó. Nếu Object không có property phù hợp, công cụ JavaScript sẽ tìm property đó trên prototype của Object
### The prototype chain
1 prototype của 1 Object là 1 Object khác. Mọi Object đều có prototype của mình (như 1 chuỗi - chain) cho đến cấp cao nhất là Object.prototype có prototype là null.

![image](https://user-images.githubusercontent.com/97771705/231651153-886d969b-5230-4ec0-b318-da4e64bb2c17.png)

Điều quan trọng là các Object kế thừa các property không chỉ từ prototype trực tiếp của chúng mà còn từ tất cả các Object phía trên chúng trong prototype chain. (ví dụ Object username kế thừa property và method của String.prototype và Object.prototype)
### Accessing an object's prototype using __proto__
Mỗi Object có một property đặc biệt mà bạn có thể sử dụng để truy cập prototype của nó, được gọi là prototype property: `__proto__`

![image](https://user-images.githubusercontent.com/97771705/231652247-dc1d9605-25be-4fed-a43b-b607680e85c3.png)

Cần phân biệt Prototype Object và Property prototype của 1 Object. 

Property prototype của 1 Object là invisible(vô hình).

![image](https://user-images.githubusercontent.com/97771705/231651771-39c9a2a3-3cb0-4cbe-b7bd-d670ae45b306.png)

### Modifying prototypes
Mặc dù nó thường được coi là một cách làm không tốt, nhưng vẫn có thể sửa đổi các built-in prototype của JavaScript giống như bất kỳ Object nào khác. Điều này có nghĩa là các nhà phát triển có thể tùy chỉnh hoặc ghi đè hành vi của các built-in method và thậm chí thêm các method mới để thực hiện các thao tác hữu ích.
# 2. Start reading
# **What is prototype pollution?**
Prototype pollution là một lỗ hổng JavaScript cho phép kẻ tấn công thêm các property tùy ý vào các global prototype Object, sau đó có thể được kế thừa bởi các Object do người dùng xác định.
## How do prototype pollution vulnerabilities arise?
Prototype pollution vulnerabilities thường phát sinh khi một hàm JavaScript hợp nhất theo cách đệ quy một Object chứa các property do người dùng kiểm soát vào một existing Object mà không sanitizing các key trước. Điều này có thể cho phép kẻ tấn công đưa vào một property bằng key như __proto__, cùng với các property lồng nhau tùy ý.

![image](https://user-images.githubusercontent.com/97771705/231929594-1865fc3d-3238-427a-b975-bb74ede7b18f.png)


Thao tác hợp nhất có thể gán các property lồng nhau cho Prototype Object thay vì chính target Object. Do đó, kẻ tấn công có thể pollute Prototype bằng các property chứa harmful values, sau đó có thể được sử dụng bởi application theo cách nguy hiểm.

Để khai thác cần 3 thành phần:
+ A prototype pollution source: Đây là bất kỳ đầu vào nào cho phép bạn poison prototype object bằng các property tùy ý.
+ A sink: một JavaScript function hoặc phần tử DOM cho phép thực thi mã tùy ý
+ An exploitable gadget: Đây là bất kỳ property nào được đưa vào sink mà không được filtering hoặc sanitization thích hợp.
## Prototype pollution sources
Prototype pollution sources là bất kỳ đầu vào nào do người dùng kiểm soát cho phép bạn thêm các property tùy ý vào các prototype object. Các nguồn phổ biến nhất như sau:

+ URL thông qua query hoặc fragment string (hash)
+ JSON-based input
+ Web messages
### Prototype pollution via the URL
`https://vulnerable-website.com/?__proto__[evilProperty]=payload`

![image](https://user-images.githubusercontent.com/97771705/231928968-1e393dcc-a324-46e4-b9ac-07a104af7847.png)

`targetObject.__proto__.evilProperty = 'payload';`

Đôi khi JS coi `__proto__` như một getter của prototype. Kết quả là, evilPropertyđược gán cho prototype object được trả về thay vì chính target object.
### Prototype pollution via JSON input
User-controllable object thường được lấy từ một JSON string bằng JSON.parse() method. JSON.parse() cũng coi bất kỳ key nào trong JSON Object là một chuỗi tùy ý, bao gồm cả `__proto__`.
```
{
    "__proto__": {
        "evilProperty": "payload"
    }
}
```
## Prototype pollution sinks
Prototype pollution sink về cơ bản chỉ là một JavaScript function hoặc phần tử DOM mà bạn có thể truy cập thông qua prototype pollution, cho phép bạn thực thi các lệnh hệ thống hoặc JavaScript tùy ý
## Prototype pollution gadgets
Một gadget cung cấp một phương tiện để biến lỗ hổng prototype pollution thành một khai thác thực tế. Đây là bất kỳ property nào:

+ Được ứng dụng sử dụng theo cách không an toàn, chẳng hạn như chuyển nó vào sink mà không được filter hoặc sanitization thích hợp.
+ Attacker có thể kiểm soát thông qua prototype pollution. Nói cách khác, Object phải có khả năng kế thừa malicious version của property được add vào prototype bởi attacker.

Một property không thể là một gadget nếu nó được xác định trực tiếp trên chính Object đó. Trong trường hợp này, phiên bản property riêng của Object được ưu tiên hơn bất kỳ phiên bản độc hại nào mà bạn có thể thêm vào prototype. Các trang web mạnh mẽ cũng có thể đặt prototype của Object thành null, điều này đảm bảo rằng Object hoàn toàn không kế thừa bất kỳ property nào.
# Client-side prototype pollution vulnerabilities
## Finding client-side prototype pollution sources manually
Các bước:
1. Thử thêm 1 property thông qua URL fragment hay JSON input 
```
vulnerable-website.com/?__proto__[foo]=bar
```
2. Tại Console, kiểm tra Object.prototype xem đã thành công pollute property chưa
3. `?__proto__[foo]=bar` không được thì xài phương án tương tự như `?__proto__.foo=bar`
4. Lặp lại cho tường source 
## Finding client-side prototype pollution sources using DOM Invader
Xài tool :)
## Finding client-side prototype pollution gadgets manually
Lú lém :(
## Finding client-side prototype pollution gadgets using DOM Invader
Xài tool :)
## Prototype pollution via the constructor
1 trong những cách để chống classic Prototype pollution là loại bỏ mọi property có khóa `__proto__` khỏi user-controlled objects trước khi merging chúng.

Nhưng cách này chưa an toàn. Trừ khi prototype được set là `null`, mọi JS Object đều có constructor property chứa tham chiếu đến constructor function tạo ra nó. Mà constructor function lại có prototype property trỏ đến prototype được gán cho bất kỳ Object nào tạo bởi constructor function này. Do đó, bạn cũng có thể truy cập prototype của bất kỳ Object nào 

-> `targetObject.constructor.prototype` tương đương với `targetObject.__proto__`

![image](https://user-images.githubusercontent.com/97771705/231935530-73fe8657-2054-4805-a6f7-8ee2050301b1.png)

## Bypassing flawed key sanitization
Website cũng có cách phòng chống khác là sanitization các property key trước khi merge chúng vào 1 exitsting Object. Nhưng đôi lúc xảy ra lỗi không sanitization chuỗi đầu vào theo cách đệ quy
```
vulnerable-website.com/?__pro__proto__to__.gadget=payload
```
## Prototype pollution in external libraries
## Prototype pollution via browser APIs
### Prototype pollution via fetch()
### Prototype pollution via Object.defineProperty()



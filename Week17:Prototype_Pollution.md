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

![image](https://user-images.githubusercontent.com/97771705/231975621-acdaa109-ee74-4261-ab9d-d9fb55288f70.png)

![image](https://user-images.githubusercontent.com/97771705/231975038-befb92af-408b-43e3-8b58-39b5f1322186.png)

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
prototype pollution gadget có thể xuất hiện trong các thư viện của bên thứ ba được ứng dụng nhập vào.
## Prototype pollution via browser APIs
### Prototype pollution via fetch()
API Fetch cung cấp một cách đơn giản để các nhà phát triển kích hoạt các HTTP request bằng JavaScript. Phương thức fetch() chấp nhận hai đối số:
+ URL mà bạn muốn gửi request.
+ Một Object tùy chọn cho phép bạn kiểm soát các phần của request, chẳng hạn như method, headers, body parameters,...

![image](https://user-images.githubusercontent.com/97771705/231939445-f9815c74-96e9-41b8-b43f-108f6700fe5f.png)

Như ví dụ trên đã xác định method và body property. Nhưng có thể còn có các property chưa xác định(như headers) nhưng trong request vẫn sử dụng nó. 

Trong trường hợp này, nếu attacker có thể tìm thấy một source phù hợp, chúng có khả năng polute Object.prototype bằng property headers. Điều này sau đó có thể được kế thừa bởi Object tùy chọn được truyền vào fetch() và sau đó được sử dụng để tạo request.

pollute Object.prototype -> option Object của fetch() -> truyền vào gadget -> thực thi code

![image](https://user-images.githubusercontent.com/97771705/231941561-eaee09c3-156a-43ba-9a28-0ac7d7a6904d.png)

![image](https://user-images.githubusercontent.com/97771705/231942319-e88dbc7d-ae35-4147-b2ef-264f8d049f88.png)

```
?__proto__[headers][x-username]=<img/src/onerror=alert(1)>
```

Bạn có thể sử dụng kỹ thuật này để kiểm soát bất kỳ property không xác định nào của Object tùy chọn được chuyển đến fetch()
### Prototype pollution via Object.defineProperty()
Object.defineProperty() xác định một property mới trực tiếp trên một Object hoặc sửa đổi một existing property trên một Object và trả về Object.
Object.defineProperty() method cho phép bạn set non-configurable, non-writable property trực tiếp trên Object bị ảnh hưởng
```
Object.defineProperty(vulnerableObject, 'gadgetProperty', {
    configurable: false,
    writable: false
})
```
Tuy nhiên, giống như fetch(), Object.defineProperty() có object tùy chọn, gọi là descriptor

![image](https://user-images.githubusercontent.com/97771705/231943566-3cffa8d0-0adb-4d65-b4ee-31231566b648.png)

Ta có thể pollute Object.prototype bằng `value` property. Sau đó nó sẽ được descriptor của Object.defineProperty kế thừa. Từ đó gán cho gadget -> thực thi code của attacker :)
# Server-side prototype pollution
## Why is server-side prototype pollution more difficult to detect?
+ Không có quyền truy cập mã nguồn
+ Thiếu công cụ dành cho nhà phát triển
+ Có thể gây ra Sự cố DoS
+ Pollution tồn tại lâu dài khi test
## Detecting server-side prototype pollution via polluted property reflection
Phương thức hasOwnProperty() trả về một giá trị boolean cho biết liệu đối tượng có thuộc tính được chỉ định làm thuộc tính riêng của nó hay không.

![image](https://user-images.githubusercontent.com/97771705/231979034-db690ede-00a6-4ad3-95b5-6e7b0f170e26.png)

Một vòng lặp for...in JavaScript lặp lại trên tất cả các property có thể đếm được của một object, bao gồm cả những property mà nó đã kế thừa qua prototype chain.

Nếu ứng dụng include các property được trả về trong response, điều này có thể cung cấp một cách đơn giản để thăm dò server-side prototype pollution

Sử dụng POST, có 1 prototype như sau
```
"__proto__":{
        "foo":"bar"
    }
```

![image](https://user-images.githubusercontent.com/97771705/231983779-20236a8e-df52-4be0-a024-db9b6b21d26a.png)

## Detecting server-side prototype pollution without polluted property reflection
Nhưng trong hầu hết mọi trường hợp, ngay cả khi chúng ta đã pollute thành công, sẽ không có bất kì property nào được reflect trong response. Vậy làm thế nào để kiểm tra xem phần inject của mình có hoạt động hay không?

Một cách tiếp cận là thử tiêm các thuộc tính phù hợp với các tùy chọn cấu hình tiềm năng cho máy chủ. Sau đó, bạn có thể so sánh hành vi của máy chủ trước và sau khi tiêm để xem liệu thay đổi cấu hình này có hiệu lực không

3 kỹ thuật là:
+ Status code override
+ JSON spaces override
+ Charset override

Các injection này là non-detructive, nhưng vẫn gây ra sự thay đổi nhất quán và đặc biệt trong hành vi của server khi thành công.
### Status code override
Server-side JavaScript framework như Express cho phép các nhà phát triển đặt HTTP response status tùy chỉnh. Trong trường hợp có lỗi, JavaScript server có thể đưa ra  HTTP response chung, nhưng include một error object ở định dạng JSON trong body. Đây là một cách cung cấp thêm chi tiết về lý do xảy ra lỗi, điều này có thể không rõ ràng từ default HTTP status.

![image](https://user-images.githubusercontent.com/97771705/231995698-04af8ae0-8ddf-4362-a894-04ec96c50c75.png)

Các bước check:
+ Tạo ra lỗi và ghi nhớ default status code 
+ Tại 1 request mới(không chủ ý tạo lỗi), Pollute bằng 1 `status` property
```
"__proto__": {
    "status":555
}
```
+ Giờ tiếp tục tạo ra lỗi và xem status code đã được override hay chưa

### JSON spaces override
Express framework có  `json spaces` option, dùng để config khoảng trắng được sử dụng để thụt lề bất kỳ dữ liệu JSON nào trong response.

Nếu bạn có quyền truy cập vào bất kỳ loại JSON response nào, bạn có thể thử làm prototype pollution bằng thuộc tính json spaces của riêng mình, sau đó gửi lại request có liên quan để xem liệu độ thụt lề trong JSON có tăng tương ứng hay không.

ví dụ 
```
"__proto__":{
"json spaces":10
}
```

![image](https://user-images.githubusercontent.com/97771705/232000039-3fc70243-376e-4542-93f5-6057cdfe9722.png)

### Charset override
Content-Type header có thể không chứa charset attribute, 
Thông thường charset là UTF-8, nhưng ta có thể override nó thành UTF-7 hoặc charset khác
```
"__proto__":{
        "content-type": "application/json; charset=utf-7"
    }
```
với `"role":"+AGYAbwBv-"` phần response sẽ trả về  `"role":"foo"` -> thành công trong việc pollute
## Scanning for server-side prototype pollution sources
## Bypassing input filters for server-side prototype pollution
Websites thường chống prototype pollution vulnerabilities bằng cách filtering key như `__proto__`, tuy nhiên điều này có thể được bypasa bằng cách sử dụng constructor property hay Obfuscate

Node application cũng có thể delete hoặc disable `__proto__` bằng cách sử dụng commad-line flag như --disable-proto=delete hoặc --disable-proto=throw. Nhưng điều này cũng có thể bypass thông qua constructor technique.  
## Remote code execution via server-side prototype pollution
### Identifying a vulnerable request
Có một số khả năng thực thi lệnh chìm trong Node, nhiều trong số đó xảy ra trong modul child_process. Chúng thường được gọi bởi một request xảy ra không đồng bộ với request mà bạn có thể làm pollution prototype ngay từ đầu. Do đó, cách tốt nhất để xác định các request này là làm polluting the prototype bằng payload kích hoạt tương tác với Burp Collaborator khi được gọi.

### Remote code execution via child_process.fork()
### Remote code execution via child_process.execSync()

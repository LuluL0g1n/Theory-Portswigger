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
Mỗi Object có một property đặc biệt mà bạn có thể sử dụng để truy cập prototype của nó, được gọi là prototype property: __proto__

![image](https://user-images.githubusercontent.com/97771705/231652247-dc1d9605-25be-4fed-a43b-b607680e85c3.png)

Cần phân biệt Prototype Object và Property prototype của 1 Object. 

Property prototype của 1 Object là invisible(vô hình).

![image](https://user-images.githubusercontent.com/97771705/231651771-39c9a2a3-3cb0-4cbe-b7bd-d670ae45b306.png)

### Modifying prototypes

# 2. Start reading
# **What is prototype pollution?**
## How do prototype pollution vulnerabilities arise?
## Prototype pollution sources
### Prototype pollution via the URL
### Prototype pollution via JSON input
## Prototype pollution sinks
## Prototype pollution gadgets
### Example of a prototype pollution gadget


# 1. XSS là gì
+ XSS là kĩ thuật tấn công để chèn các script nguy hiểm vào source code ứng dụng web, mục đích thường là chiếm lấy phiên đăng nhập của người dùng.
# 2. Các loại XSS

## 2.1. Reflected XSS: 
+ Script nguy hiểm được chèn trực tiếp vào response của user, hacker cần phải gửi link độc hại này cho user và lừa user nhấp vào link. Payload được thực hiện trong cùng một request và response nên còn được gọi là first-order XSS.
+ Phát sinh khi ứng dụng nhận dữ liệu trong HTTP request và include dữ liệu đó trong response ngay lập tức theo cách không an toàn.

+ Tác động
> Hacker có thể xem, sửa, xóa dữ liệu của victim

> Tấn công đến người dùng khác, bắt nguồn từ tài khoản victim

+ Phương pháp tấn công phổ biến
> Gửi email, tweet, message

> Đặt link trên trang web mà hacker kiểm soát rồi lừa người dùng nhấp vào

+ Cách kiểm tra lỗ hổng : sử dụng trình quét của burpsuit
+ Cách kiểm tra lỗ hổng thủ công: theo các bước sau
> B1: Kiểm tra tất cả đầu vào: Xem có thể nhập dữ liệu tại những đâu

> B2: Gửi giá trị ngẫu nhiên xem có reflected trong response hay không

> B3: Xác định context mà xảy ra reflected: có thể là thẻ HTML, cũng có thể là đoạn javacscript

> B4: Gửi payload thường: <script> alert()</script
  
> B5: Gửi các payload thay thế tương đương

> B6: Nếu tìm thấy payload hoạt động, thử test trên browser

+ Sự khác nhau giữa Reflect XSS và Store XSS: Reflect XSS ứng dụng lấy input từ HTTP request và nhúng input đó vào response ngay lập tức theo 1 cách không an toàn(không mã hóa, làm sạch gì cả). Store XSS ứng dụng store input đó và nhúng vào response sau.
+ Self XSS: chỉ được kích hoạt khi nạn nhân dán XSS payload trên chính browser của họ. Về cơ bản nó giống như một cuộc tấn công kĩ nghệ xã hội.

+ XSS context: Bao gồm (trong thẻ html, trong attribute thẻ html, trong js, trong js template literal)
> Vị trí trong phản hồi xuất hiện dữ liệu có thể kiểm soát của kẻ tấn công

> Mọi xác thực đầu vào hoặc quá trình xử lý khác đang được ứng dụng thực hiện trên dữ liệu đó.

### 2.1.1 Bài tập
+ Một số bài như **Reflected XSS into HTML context with most tags and attributes blocked**, **Reflected XSS into HTML context with all tags blocked except custom ones** yêu cầu phải sử dụng Burp Intruder để tìm kiếm tag, attribute và event không bị chặn, dữ liệu tên các thẻ nằm ở **XSS Cheat Sheet** của Portswwiger
+ Đôi khi một số trang chỉ cho phép up ảnh .jpg hoặc .png vẫn có thể cho phép sử dụng ảnh svg
+ Nếu như XSS context nằm trong thuộc tính href của thẻ neo, sử dụng giao thức giả javascript
![image](https://user-images.githubusercontent.com/97771705/216812102-15dcfd66-b802-46ec-80ff-61a4e0267e79.png)
+ Một số thẻ bị mã hóa dấu ngoặc nhọn hoặc những thẻ không tự động kích hoạt event như canonical tags vẫn cho phép thêm attribute, khi đó ta sử dụng attribute accesskey để kích hoạt event.
+ Khi XSS context tại JS, có thể sử dụng phương terminaling script(đóng script và thực thi),break out JS string(tạm thời thoát ra khỏi JS string để thực thi), mã hóa HTML hoặc sử dụng template literal JS (``)
+ Khi XSS context tại JS template literal, không cần phải terminal script, chỉ cần sd cú pháp ${...}

### 2.1.2 Một số thẻ, event đáng chú ý
+ Object Location : được sử dụng để lấy địa chỉ
 trang hiện tại
+ Event onfocus: xảy ra khi một phần tử được lấy tiêu điểm, cần sự kích hoạt của người dùng như click chuột hoặc nhấn tab
![image](https://user-images.githubusercontent.com/97771705/216810811-2c7ba076-66e1-49b7-a915-a799194649ce.png)
+ Event onblur: xảy ra khi 1 phần tử mất tiêu điểm
+ ![image](https://user-images.githubusercontent.com/97771705/216810884-04788f18-0e81-44ae-91be-3f261cfa48b1.png)
+ Attribute tabindex: chỉ định thứ tự tab của một phần tử ,cho phép developer làm cho các HTML elements có thể được đặt focus
+ Thẻ <svg: là thẻ sử dụng XML,miêu tả các hình ảnh đồ họa vector dạng 2 chiều, hoạt hình và tĩnh.
+ Canonical URL (hay còn gọi là Rel Canonical) là thành phần HTML nhằm khai báo URL gốc của trang bị trùng lặp nội dung với công cụ tìm kiếm. Sử dụng thẻ Canonical trong trường hợp nội dung bị Duplicate hoặc giống nhau trên nhiều URL.
+ Attribute accesskey: chỉ định một phím tắt để active/focus một phần tử.
+ Template literal JS: là các chuỗi ký tự cho phép các biểu thức JavaScript được nhúng. Các biểu thức nhúng thường được nối vào văn bản xung quanh và được xác định bằng ${...}
## 2.2. Stored XSS: 
+ Hacker chèn script nguy hiểm vào trang web lỗi, script sẽ được lưu lại trong ứng dụng web. Bất cứ khi nào user truy cập vào trang web có chứa đoạn script đó, script sẽ được thực thi. Chính vì attack xảy ra qua 2 bước như vậy nên loại này còn được gọi là second-order XSS.
+ Store XSS chỉ cần lưu script trên app và ngồi đợi user truy cập.
+ Store XSS chỉ ảnh hướng đến người dùng đã login 

## 2.3. DOM based XSS: 
+ Khá giống với Reflected XSS, tuy nhiên script của hacker sẽ không được nhúng trực tiếp vào ứng dụng web mà thông qua DOM (Document Object Model) và không giống như 2 loại XSS trên, mã độc sẽ được thực thi ngay khi xử lý phía client mà không thông qua server.
+ Các lỗ hổng XSS dựa trên DOM thường phát sinh khi JavaScript lấy dữ liệu từ một source có thể kiểm soát được của kẻ tấn công, chẳng hạn như URL và chuyển dữ liệu đó tới phần sink hỗ trợ thực thi mã động, chẳng hạn như eval()hoặc innerHTML

### 2.3.1. Bài tập

### 2.3.2 Thuật ngữ 

+ Source là thuộc tính JavaScript chấp nhận dữ liệu có khả năng bị kẻ tấn công kiểm soát. VD URL, attribute location, database....
+ Sink là một hàm JavaScript hoặc đối tượng DOM nguy hiểm tiềm ẩn có thể gây ra các tác động không mong muốn nếu dữ liệu do kẻ tấn công kiểm soát được chuyển đến nó. VD eval(), document.body.innerHTML, postMessage,document.write, document.domain...

# 3. Một số cách bypass

+ Thay đổi chữ hoa thường. VD: <script -> <ScRipt , <SCRIPT hoặc nhiều mẫu khác
+ Thay đổi thẻ: có rất nhiều thẻ thực thi được javascript, tham khảo trên github https://github.com/swisskyrepo/PayloadsAllTheThings

# 4. Exploit XSS
+ Steel cookie 
  ![image](https://user-images.githubusercontent.com/97771705/216878473-dda56192-401b-433c-9ce2-caa91fd3f809.png)
+ Steel password
+ Khai thác XSS để thực hiện CSRF
  ![image](https://user-images.githubusercontent.com/97771705/216879037-4b306939-961a-49ad-9ce7-a4bd3404a21f.png)
# 5. More
  https://portswigger.net/web-security/cross-site-scripting/dangling-markup
  https://portswigger.net/web-security/cross-site-scripting/contexts/client-side-template-injection
+ Sandbox là một kỹ thuật giúp cô lập các ứng dụng, giúp bảo vệ và không cho các phần mềm độc hại xâm nhập vào máy tính, điện thoại để hạn chế hỏng hệ thống máy hoặc rò rỉ các thông tin cá nhân.Sandbox sẽ giới hạn chức năng của một đoạn mã nào đó và cấp quyền cho chúng chỉ được hiện một số chức năng nhất định, không cho nó thực hiện những chức năng khác gây nguy hại đến máy tính của bạn. 
+ Kỹ thuật khi XSS without HTML: Client-side template injection :Xảy ra khi app nhúng template framework của client vào web. Khi nhúng sẽ thực thi tất cả các template, kể cả template độc hại
# 6. Cách ngăn chặn
+ Lọc dữ liệu đầu vào
+ Mã hóa dữ liệu đầu ra
+ Lập whitelist
+ Sử dụng template engine
+ Trong PHP: sử dụng hàm htmlentites, hàm jsescape 
+ How to prevent XSS client-side in JavaScript: tự tạo bộ encode HTML
+ Sử dụng CSP: CSP là một cơ chế bảo mật của trình duyệt nhằm giảm thiểu XSS và một số cuộc tấn công khác. Nó hoạt động bằng cách hạn chế các tài nguyên (chẳng hạn như script và image) mà một trang có thể load và hạn chế liệu một trang có thể frame bởi các trang khác hay không.
 Để kích hoạt CSP, một respone cần bao gồm HTTP response header Content-Security-Policy
![image](https://user-images.githubusercontent.com/97771705/216882445-4562a070-81e8-4703-a063-c465e480cb6c.png)
![image](https://user-images.githubusercontent.com/97771705/216882477-18c020ee-1e3a-45bb-9f29-4ed511436787.png)
![image](https://user-images.githubusercontent.com/97771705/216882498-e0de4b67-3ffd-49b6-a0f0-399a7311a41c.png)

![image](https://user-images.githubusercontent.com/97771705/216882689-787aaf09-e845-496c-895c-10c8c8c095ca.png)

 
  

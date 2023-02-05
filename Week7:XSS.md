1. XSS là gì
+ XSS là kĩ thuật tấn công để chèn các script nguy hiểm vào source code ứng dụng web, mục đích thường là chiếm lấy phiên đăng nhập của người dùng.
2. Các loại XSS

2.1. Reflected XSS: 
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
> B4: Gửi payload thường: <script> alert()</script>
> B5: Gửi các payload thay thế tương đương
> B6: Nếu tìm thấy payload hoạt động, thử test trên browser
2.2. Stored XSS: 
+ Hacker chèn script nguy hiểm vào trang web lỗi, script sẽ được lưu lại trong ứng dụng web. Bất cứ khi nào user truy cập vào trang web có chứa đoạn script đó, script sẽ được thực thi. Chính vì attack xảy ra qua 2 bước như vậy nên loại này còn được gọi là second-order XSS.

2.3. DOM based XSS: 
+ Khá giống với Reflected XSS, tuy nhiên script của hacker sẽ không được nhúng trực tiếp vào ứng dụng web mà thông qua DOM (Document Object Model) và không giống như 2 loại XSS trên, mã độc sẽ được thực thi ngay khi xử lý phía client mà không thông qua server.

3. Một số cách bypass

+ Thay đổi chữ hoa thường. VD: <script -> <ScRipt , <SCRIPT hoặc nhiều mẫu khác
+ Thay đổi thẻ: có rất nhiều thẻ thực thi được javascript, tham khảo trên github https://github.com/swisskyrepo/PayloadsAllTheThings


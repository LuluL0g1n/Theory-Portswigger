1. XSS là gì
> XSS là kĩ thuật tấn công để chèn các script nguy hiểm vào source code ứng dụng web, mục đích thường là chiếm lấy phiên đăng nhập của người dùng.
2. Các loại XSS

2.1. Reflected XSS: 
> Script nguy hiểm được chèn trực tiếp vào response của user, hacker cần phải gửi link độc hại này cho user và lừa user nhấp vào link. Payload được thực hiện trong cùng một request và response nên còn được gọi là first-order XSS.

2.2. Stored XSS: 
> Hacker chèn script nguy hiểm vào trang web lỗi, script sẽ được lưu lại trong ứng dụng web. Bất cứ khi nào user truy cập vào trang web có chứa đoạn script đó, script sẽ được thực thi. Chính vì attack xảy ra qua 2 bước như vậy nên loại này còn được gọi là second-order XSS.

2.3. DOM based XSS: 
> Khá giống với Reflected XSS, tuy nhiên script của hacker sẽ không được nhúng trực tiếp vào ứng dụng web mà thông qua DOM (Document Object Model) và không giống như 2 loại XSS trên, mã độc sẽ được thực thi ngay khi xử lý phía client mà không thông qua server.

3. Một số cách bypass

+ Thay đổi chữ hoa thường. VD: <script -> <ScRipt , <SCRIPT hoặc nhiều mẫu khác
+ Thay đổi thẻ: có rất nhiều thẻ thực thi được javascript, tham khảo trên github https://github.com/swisskyrepo/PayloadsAllTheThings


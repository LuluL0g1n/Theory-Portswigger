# 1. CSRF
+ Là một lỗ hổng bảo mật web cho phép kẻ tấn công khiến người dùng thực hiện các hành động mà họ không có ý định thực hiện như thay đổi địa chỉ email, đổi mật khẩu , chuyển tiền.
+ Để một cuộc tấn công CSRF có thể thực hiện được, phải có ba điều kiện chính:
> Có 1 hành động mà hacker quan tâm (vd change password, cấp quyền cho user)

> Ứng dụng chỉ dựa vào sesion cookie để xác định người dùng đã thực hiện yêu cầu  

> Request không chứa các tham số mà hacker không biết hoặc không thể đoán. VD request change password cần phải biết mật khẩu hiện tại. 

![image](https://user-images.githubusercontent.com/97771705/216905098-a27c0228-b69a-4937-b621-49c49efbf8e7.png)
+ CSRF-token đôi khi bảo vệ trang web khỏi Reflect XSS nhưng không thể bảo vệ khỏi Store XSS. Quá trình bve: nếu 1 trang web check CSRF-token mỗi khi gửi request, thì lỗi Reflect XSS đơn giản không thể thành công
![image](https://user-images.githubusercontent.com/97771705/216906353-ee1366e9-bfcd-4ad5-b7eb-84dda10eb0d2.png)
![image](https://user-images.githubusercontent.com/97771705/216906385-047533d9-ec87-4bd8-9f27-16a59f393791.png)

https://portswigger.net/web-security/csrf/xss-vs-csrf

+ Một ví dụ đơn giản về CSRF: 1 user đăng nhập trên example.com rồi thoát ra không log out. user đó lại truy cập vào 1 trang web do hacker kiểm soát. hacker có lưu 1 đoạn HTML PoC nhằm đổi email user trên trang example.com. Khi truy cập thì ngay lập tức kích hoạt đoạn PoC đó và đổi email user(do user vẫn còn login)

![image](https://user-images.githubusercontent.com/97771705/216908789-84ff0843-3a0c-401c-8228-0af42b8ff4d6.png)

# 2. Cách phòng chống
## 2.1 CSRF token
+  là một giá trị duy nhất, bí mật và không thể đoán trước được tạo bởi ứng dụng phía máy chủ và được chia sẻ với máy khách. Khi đưa ra yêu cầu thực hiện một hành động nhạy cảm, chẳng hạn như gửi biểu mẫu, ứng dụng khách phải bao gồm mã thông báo CSRF chính xác. Nếu không, máy chủ sẽ từ chối thực hiện hành động được yêu cầu.
+  Một số phương pháp bypass
> Đôi khi server chỉ xác thực token với method POST mà không xác thực GET -> sử dụng method GET

> Một số app sẽ xác thực token, khi có token trong request -> không gửi token kèm request nữa

> Một số app xây dựng token không ràng buộc với session, tức là chỉ kiểm tra xem token này có trùng trong list token của app hay không -> lấy token ở acc của hacker rồi sử dụng cho victim
![image](https://user-images.githubusercontent.com/97771705/216913030-3b058751-6f94-4f97-94b4-ce724bb72ec3.png)

> Một biến thể của phần trên: có csrfKey tại header Cookies và csrf trong request. App sẽ check 2 cái nài có cùng 1 bộ không, và không có ràng buộc với Cookie session. Bộ token của người này có thể sử dụng cho người khác. Để bypass, nhập input sao cho tạo được header Set-Cookie đứng ở 1 dòng riêng

> Biến thể tiếp theo là bộ csrf chỉ có 1 giá trị (duplicate trong cả 2 token). Cách này hay được sử dụng hơn do backend không phải lưu trữ list cặp token hợp lệ mà chỉ cần check 2 cái xem có bằng nhau không. Nhưng nếu có nơi để nhập cài đặt cookie thì vẫn là lỗ hổng. Sử dụng cách Set-Cookie như bài trên.

## 2.2 Cookie SameSite
+ 1 site được định nghĩa là top-level domain (TLD) như .com, .net; cộng thêm domain name -> gọi là TLD+1
![image](https://user-images.githubusercontent.com/97771705/216921705-242de2b8-efbf-4db3-913b-2344901b67ed.png)

+ Sự khác biệt giữa 1 site và 1 origin: phạm vi
> 1 Site gồm nhiều domain name, trong khi origin chỉ có 1

> 2 URL được coi là same site nếu chúng cùng scheme và cùng TLD +1 (TLD khác eTLD. vd eTLD là .co.uk)

> 2 URL được coi là same origin nếu chúng cùng scheme, domain name và port (đôi khi port được suy ra từ scheme)
![image](https://user-images.githubusercontent.com/97771705/216922377-9ed21466-2e41-4f74-b898-3b002c005591.png)
![image](https://user-images.githubusercontent.com/97771705/216922600-99e78994-7e12-45e6-82d4-1439d55f61cc.png)

+ 3 mức hạn chế của Same Site: Strict ,Lax, None

Nếu một cookie được đặt với SameSite=Strict, browser sẽ không gửi nó trong bất kỳ cross-site request nào. Nói một cách đơn giản, điều này có nghĩa là nếu target site cho request không khớp với trang hiện được hiển thị trên thanh địa chỉ của trình duyệt, nó sẽ không include cookie. -> tránh được việc sửa đổi dữ liệu hay thực hiện các hành động kém an toàn 

An toàn nhất, tuy nhiên lại ảnh hưởng không nhỏ tới người dùng
> Bypass thông qua client-side redirect

>Bypass thông qua sibling domain

Lax thì nới lỏng hơn: cho phép gửi cross-site request, tuy nhiên vẫn có 2 điều kiện:
1. Sử dụng phương thức GET
2. Request cuất phát từ top-level navigation của user, chẳng hạn như click vào link
Top-level navigation thay đổi URL trong thanh địa chỉ. Các tài nguyên được tải bởi iframe, thẻ img và thẻ script không thay đổi URL trong thanh địa chỉ nên không tài nguyên nào gây ra top-level navigation.

> Đôi khi ta có thể ghi đè phương thức để đạt mục đích. Symfony hỗ trợ tham số  `_method` trong biểu mẫu, cho phép ưu tiên method -> ghi đè từ POST sang GET

> Thông qua cookies refresh: Nếu không thiết lập thủ công samesite lax thì chrome sẽ tự thiết lập. Tuy nhiên trong vòng 2 phút đầu tiên khi gửi POST top-level navigation thì Chrome không thực thi hạn chế này để tránh vi phạm SSO. -> lợi dụng khoảng thời gian này để khai thác csrf

None thì bỏ qua hoàn toàn hạn chế của Samesite. Có những lý do chính đáng để vô hiệu hóa SameSite, chẳng hạn như khi cookie được dự định sử dụng từ ngữ cảnh của bên thứ ba và không cấp cho người mang quyền truy cập vào bất kỳ dữ liệu hoặc chức năng nhạy cảm nào. Cookie theo dõi là một ví dụ điển hình.
 
=> Biện pháp an toàn là sử dụng Lax-by-default 
## 2.3 Referer HTML header
+ Một số ứng dụng sử dụng HTTP header Referer để cố gắng bảo vệ chống lại các cuộc tấn công CSRF, thông thường bằng cách xác minh rằng yêu cầu bắt nguồn từ application's own domain
+ Referer header: là 1 request header tùy ý chứa URL của trang web được liên kết với tài nguyên đang được yêu cầu.
![image](https://user-images.githubusercontent.com/97771705/216938481-2b556cda-9e63-4863-b64d-f7bd96bc299f.png)

![image](https://user-images.githubusercontent.com/97771705/216938625-f1925b48-f009-44c3-b817-8ac2ba61acc3.png)

![image](https://user-images.githubusercontent.com/97771705/216941682-1740d48c-306c-4c38-a5e6-0adc18265059.png)

+ Referrer-Policy: unsafe-url cho phép gửi origin, path, chuỗi truy vấn khi thực hiện bất kì request nào, không quan tâm đến vấn đề bảo mật
# 3. Cách ngăn chặn
+ Tạo csrf token: Không thể đoán trước,gắn với phiên người dùng và được xác thực nghiêm ngặt trong mọi trường hợp trước khi hành động liên quan được thực hiện.
![image](https://user-images.githubusercontent.com/97771705/216943071-f9eac513-72d7-4b07-bcc6-9ec409817665.png)

![image](https://user-images.githubusercontent.com/97771705/216943321-1f6133a0-91aa-426b-8f1c-e8b99cd45dbe.png)

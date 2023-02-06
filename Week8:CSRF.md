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
## 2.3 Referer HTML header

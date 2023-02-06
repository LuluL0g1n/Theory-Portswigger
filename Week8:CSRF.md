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
+ 

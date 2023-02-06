# I. CORS
## 1. CORS
![image](https://user-images.githubusercontent.com/97771705/216949521-d9473806-afb2-4ecc-9d04-e587c08ffe9d.png)
+ Same-Origin: hạn chế các tập lệnh trên một origin truy cập dữ liệu từ một origin khác.
![image](https://user-images.githubusercontent.com/97771705/216991834-c39b52a2-4650-4a8c-baaf-bd61ed43d982.png)
https://portswigger.net/web-security/cors/same-origin-policy
+ CORS sử dụng một bộ tiêu đề HTTP xác định origin web đáng tin cậy và các thuộc tính liên quan, chẳng hạn như liệu quyền truy cập được xác thực có được phép hay không. Chúng được kết hợp trong header exchange giữa browser và cross-origin website mà beowser đang cố truy cập
+ CORS là sự nới lỏng SOP
+ Access-Control-Allow-Origin header được include trong response từ 1 website đối với request từ website khác và xác định origin được cho phép của request
+ Trình duyệt web so sánh Access-Control-Allow-Origin với the requesting website's origin và cho phép truy cập vào phản hồi nếu chúng khớp.
+ ACAO = * là rất nguy hiểm, nên 1 số browsers tự tạo ACAO dựa trên origin của máy khách
+ Pre-flight checks: https://portswigger.net/web-security/cors/access-control-allow-origin
![image](https://user-images.githubusercontent.com/97771705/217006388-cccad8ef-7207-4152-a05e-db5038551d59.png)

# II. Clickjacking
## Clickjacking
+  Cuộc tấn công này khác với cuộc tấn công CSRF ở chỗ người dùng được yêu cầu thực hiện một hành động chẳng hạn như nhấp vào nút trong khi cuộc tấn công CSRF phụ thuộc vào việc giả mạo toàn bộ yêu cầu mà người dùng không biết hoặc đầu vào.
+  CSRF token không thể ngăn chặn clickjacking
+  Có thể gặp dạng bài server sử dụng cách phá khung hoặc chặn khung. Bypass bằng thuộc tính sandbox='allow-form" của iframe 
+  Ngăn chặn bằng việc sử dụng X-Frame-Option hoặc CSP
![image](https://user-images.githubusercontent.com/97771705/217013223-ca7330ab-fce9-4bfc-920d-1ae5002c747e.png)
![image](https://user-images.githubusercontent.com/97771705/217013293-aebbcf47-a99d-4ac0-8deb-4db226f8836d.png)


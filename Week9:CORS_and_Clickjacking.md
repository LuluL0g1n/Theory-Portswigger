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
# II. Clickjacking

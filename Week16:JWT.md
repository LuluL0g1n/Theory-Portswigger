# JWT attacks
## What are JWTs?
JSON web tokens (JWTs) là một định dạng được tiêu chuẩn hóa để gửi dữ liệu JSON được ký bằng mật mã giữa các hệ thống.

Thường được sử dụng nhất để gửi thông tin ("claims") về người dùng như một phần của cơ chế xác thực, xử lý phiên và kiểm soát truy cập.

Tất cả dữ liệu mà server cần được lưu trữ phía client bên trong JWT.
### JWT format
Một JWT bao gồm 3 phần: header , payload và signature; phân cách nhau bởi dấu chấm

![image](https://user-images.githubusercontent.com/97771705/232182661-62899c15-d09d-4640-b13c-665bde84bbf2.png)

Header và payload của JWT là JSON object được mã hóa base64url

Header chứa các thông tin như thuật toán dùng để ký (RS256, HS256) hay token type (JWT, SAML)

Payload chứa thông tin ('claims') của người dùng 

-> hoàn toàn có thể đọc và sửa Header và Payload -> JWT phụ thuộc rất nhiều vào phần signature
### JWT signature
Về cơ bản, quá trình tạo mã signature như sau
+ B1: Phần header và payload được mã hóa base64url
+ B2: Nối cả 2 phần thành 1 chuỗi, ngăn cách bởi dấu `.`
+ B3: Xác định thuật toán. VD HS256 là HMAC + SHA256, RS256 là RSA + SHA256. Ta Hash chuối ở bước 3 với thuật toán Hash được xác định như SHA256. 
+ B4: Chuỗi sau khi hash sẽ được mã hóa bởi thuật toán còn lại.( VD RSA thì có 2 khóa: public và private key. Sử dụng private key để mã hóa, và tiết lộ public key để server giải mã. Còn HMAC thì có secret key)
+ B5: Cuối cùng nối signature vào chuỗi ở B2 với dấu `.`, ta được JWT hoàn chỉnh

Sig là phòng tuyến quan trọng nhât vì:
+ Sig được làm từ Header và Payload thì sửa 1 byte cũng sẽ dẫn đến kết quả khác nhau. 
+ Không có private key (đối vs RSA) và secret key (với HMAC) thì không thể tạo sig 
### JWT vs JWS vs JWE
Một chuỗi Signed JWT được gọi là JWS (JSON Web Signature). Trong thực tế JWT không tồn tại riêng lẻ mà nó phải là JWS hoặc JWE (JSON Web Encryption).

Nói cách khác, JWT thường là JWS hoặc JWE token. Khi mọi người sử dụng thuật ngữ "JWT", hầu như họ luôn có nghĩa là JWS token. JWE tương tự, ngoại trừ nội dung thực tế của token được encrypt thay vì chỉ được encode. (encrypt có sử dụng key, encode thì không)
## What are JWT attacks?
## What is the impact of JWT attacks?
## How do vulnerabilities to JWT attacks arise?
## Exploiting flawed JWT signature verification
### Accepting arbitrary signatures
### Accepting tokens with no signature
## Brute-forcing secret keys
### Brute-forcing secret keys using hashcat
## JWT header parameter injections
### Injecting self-signed JWTs via the jwk parameter
### Injecting self-signed JWTs via the jku parameter
### Injecting self-signed JWTs via the kid parameter
### Other interesting JWT header parameters
## JWT algorithm confusion
## How to prevent JWT attacks
### Additional best practice for JWT handling

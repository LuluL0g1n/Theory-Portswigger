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

JWT chỉ có 2 parameter là typ(type) và cty(content type) trong JOSE header.Tuy nhiên JWS có thêm alg(algorithm), kid(key id)...., cũng như JWE có enc(content encryption algorithm) và zip(Compression Algorithm)

JWT header còn được gọi là JOSE Header (Javascript Object Signing and Encryption). Cả JWS và JWE đều có JOSE header
![image](https://user-images.githubusercontent.com/97771705/232201670-203d0338-3653-42f5-aa79-218cf010c27c.png)

![image](https://user-images.githubusercontent.com/97771705/232201729-2a5eff32-cec4-4bc4-a9e3-77a9139056ee.png)

![image](https://user-images.githubusercontent.com/97771705/232201733-43c76797-b7d7-4a08-811f-7f4098d1d195.png)

Trong thực tế JWT không tồn tại riêng lẻ mà nó phải là JWS (JSON Web Signature) hoặc JWE (JSON Web Encryption).

Nói cách khác, JWT thường là JWS hoặc JWE token. Khi mọi người sử dụng thuật ngữ "JWT", hầu như họ luôn có nghĩa là JWS token. JWE tương tự, ngoại trừ nội dung thực tế của token được encrypt thay vì chỉ được encode. (encrypt có sử dụng key, encode thì không)

Phân biệt bằng `typ` parameter: nếu mà `typ`:`JWS` thì đó là JWS, `typ`:`JWT` thì đó là JWT :)

https://medium.com/oho-software/jwt-jws-v%C3%A0-jwe-cb1f0fedf61c
## What are JWT attacks?
JWT attack liên quan đến việc user gửi JWT đã được sửa đến server nhằm mục đích bỏ qua xác thực hay kiểm soát truy cập bằng cách mạo danh một người dùng khác đã được xác thực.
## What is the impact of JWT attacks?
Tác động: leo thang đặc quyền, mạo danh user khác, chiếm quyền kiểm soát tài khoản của họ.
## How do vulnerabilities to JWT attacks arise?
JWT vul thường phát sinh do lỗi xử lý JWT trong chính ứng dụng.

1 vài trường hợp 
+ Sig của JWT không được kiểm tra chính xác (bỏ qua việc kiểm tra sig, chỉ kiểm tra sig có tồn tại trên server bằng cách so sánh chứ k quan tâm đến header và payload có bị đổi không...)
+ Sig có đáng tin cậy không (mặc dù đúng nhưng có thể là sig được tạo bởi 1 bên thứ 3 nào đó)
## Exploiting flawed JWT signature verification
### Accepting arbitrary signatures
Các thư viện JWT thường cung cấp một method để xác minh token và một method khác chỉ giải mã chúng. Ví dụ: thư viện jsonwebtoken của Node.js có verify()và decode().

Đôi khi JWT chỉ bị chuyển đến `decode()` mà sau đó không đến `verrify()` -> tức là không có xác thực gì hết... một cái mã vớ vẩn sửa header với payload tùm lum cũng có thể hợp lệ.
### Accepting tokens with no signature
Header của JWT chứa `alg` parameter. Cái này xác nhận thuật toán nào được sử dụng để ký token (hay là tạo signature) như HS256, RS256 ,ES256...
```
{
    "alg": "HS256",
    "typ": "JWT"
}
```
Khi sửa 'alg':'none' tức là k có thuật toán nào được sử dụng để ký -> máy chủ chấp nhận alg này tức là kệ luôn phần signature :)
## Brute-forcing secret keys
### Brute-forcing secret keys using hashcat
xài tool hashcat :)
## JWT header parameter injections
Trong JWS, `alg` là parameter bắt buộc. Tuy nhiên có một vài parameter khác cần chú ý 
+ jwk (JSON Web Key): là một embedded JSON object đại diện cho khóa
+ jku (JSON Web Key Set URL): Cung cấp một URL từ đó các server có thể tìm nạp một bộ khóa chứa key chính xác.
+ kid (Key ID): Cung cấp ID mà server có thể sử dụng để xác định khóa chính xác trong trường hợp có nhiều khóa để chọn.
### Injecting self-signed JWTs via the jwk parameter
`jwk` header:  server có thể sử dụng để nhúng public key của họ trực tiếp vào trong token ở JWK format.
```
{
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "typ": "JWT",
    "alg": "RS256",
    "jwk": {
        "kty": "RSA", #key type
        "e": "AQAB",
        "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
        "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
    }
}
```
Server nên có 1 whitelist để xác định public key nào là hợp lệ. Nếu không, người dùng sẽ tự tạo thuật toán RSA: ký bằng private key và đưa thông thin public key lên jwk. -> việc xác thực luôn luôn đúng :)

### Injecting self-signed JWTs via the jku parameter
`jku` header cung cấp 1 URL mà tại đó có JWK Set chứa khóa.

JWK Set là 1 JSON object chứa một mảng JWK đại diện cho các khóa khác nhau

```
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```
Ta có thể chuyển jku sang 1 URL khác có chứa khóa mà ta kiểm soát.
### Injecting self-signed JWTs via the kid parameter
Thực tế là server có rất nhiều key, và `kid` parameter sinh ra để tránh gây lú.

Do các key thường được lưu trong JWK Set,  `kid`(key ID) xác định ID của key trong JWK Set. 

Tuy nhiên JWS không xác định cấu trúc cụ thể cho ID này - nó chỉ là một chuỗi tùy ý do nhà phát triển lựa chọn. Ví dụ: họ có thể sử dụng `kid` để trỏ đến một mục cụ thể trong cơ sở dữ liệu hoặc thậm chí là tên của tệp.

Lỗ hổng ở đây là ta dùng `kid` để trỏ đến 1 tệp trong hệ thống mà ta biết giá trị, sign token bằng giá trị đó. Như vậy server sẽ lấy giá trị trong tệp mà `kid` trỏ tới và giải mã -> bypass thành công

1 trong những cách đơn giản nhất là sử dụng `/dev/null` chứa giá trị null, có trong hầu hết mọi hệ thống linux
### Other interesting JWT header parameters
+ cty (Content Type): Đôi khi ta có thể thay đổi cty thành `text/xml` hoặc `application/x-java-serialized-object` để tạo attack vector cho XXE hay deserialization attacks.
+ x5c (X.509 Certificate Chain)
## JWT algorithm confusion
Cuộc tấn công này attacker buộc server phải xác minh signature của JWT bằng 1 thuật toán khác so với dự kiến. Nếu trường hợp này không được xử lý đúng cách, điều này có thể cho phép attacker giả mạo JWT hợp lệ chứa các giá trị tùy ý mà không cần biết secret signing key của server.

Dạng cơ bản là: ta có 1 JWT sử dụng RS256.
+ Ta tìm kiếm public key trên server rồi chuyển nó về dạng thích hợp (giống với định dạng secret key bên server VD như X.509 PEM)
+ Sign với key đó. Sửa đổi `alg`:"HS256"
+ Gửi JWT đi

Server tiếp tục sử dụng secret key đó để giải mã HS256. -> bypass thành công

Trong trường hợp không kiếm được public key trên server, ta có thể sử dụng các tool như jwt_forgery.py hay rsa_sign2n để tìm. Có thể kết quả trả về khi dung tool này là 1 hoặc nhiều giá trị n tiềm năng nhưng chắc chắn 1 trong số chúng là chính xác. Ta sẽ đi thử và xác nhận

Tại sao giả trị n là đủ? vì khi tạo symmetric key, ta chỉ cần giá trị n cho `k` (key) parameter trên JWT editor mà thôi
## How to prevent JWT attacks
+ white list cho `jku` parameter
+ Đảm bảo không có path traversal hoặc SQL injection thông qua `kid` paramater
+ Đảm bảo luôn xác thực signature JWT 1 cách cẩn thận, chính xác
+ Sử dụng up-to-date library cho JWT 
### Additional best practice for JWT handling
+ Đặt expiration date(ngày hết hạn) cho mọi token
+ Tránh gửi token trong URL
+ Sử dụng `aud` (audience) để chỉ định người nhận token, tránh tình trạng được sử dụng bởi website khác
+ Cho phép server thu hổi token( ví dụ khi đăng xuất)

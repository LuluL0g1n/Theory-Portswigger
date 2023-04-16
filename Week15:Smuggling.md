# HTTP request smuggling
## What is HTTP request smuggling?
HRS là một kỹ thuật can thiệp vào cách một trang web xử lý các chuỗi HTTP request nhận được từ một hoặc nhiều người dùng.

Lỗ hổng này về bản chất thường rất nghiêm trọng, cho phép attacker vượt qua các biện pháp kiểm soát bảo mật, giành quyền truy cập trái phép vào dữ liệu nhạy cảm và trực tiếp xâm phạm những người dùng ứng dụng khác.
## What happens in an HTTP request smuggling attack?
Người dùng send request tới front-end server, và sau đó nó sẽ chuyển tiếp request tới backend server.

Khi front-end server chuyển tiếp request tới back-end server, nó sẽ gửi một vài request trên cùng 1 kết nối backend network, vì nó hiệu quả (mỗi 1 request dùng 1 kết nối riêng thì cần nhiều lắm). Giao thức rất đơn giản: gửi lần lượt request, phía server nhận sẽ check header của các request đó để biết được đâu là kết thúc của 1 request, và đâu là bắt đầu của 1 request kế.

![image](https://user-images.githubusercontent.com/97771705/232237446-04a68675-82d8-41d2-8740-78b22a64e69b.png)

Nhưng front-end và back-end server phải thống nhất về ranh giới giữa 2 request. Lỗ hổng này tận dụng việc ta gửi 1 request nhưng 2 server có cách diễn giải khác nhau 

![image](https://user-images.githubusercontent.com/97771705/232237545-bdc835aa-3351-43d8-b23d-82387cada372.png)

Việc mập mờ như này làm cho phía backend tưởng rằng 1 phần cuối của attacker request từ frontend là bắt đầu 1 request mới (tạm gọi là half-request )  , bất chấp phía frontend coi cả request đó là 1 mà thôi. Như vậy request ngay phía sau sẽ bị đính vào làm 1 phần của half-request -> attack thành công
## How do HTTP request smuggling vulnerabilities arise?
Có 2 cách để chỉ định nơi request kết thúc: `Content-Length` header và `Transfer-Encoding` header
+ Content-Length: chỉ định độ dài của phần body tính bằng byte
```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```
+ Transfer-Encoding: được sử dụng để xác định rằng message body sử dụng chunked encoding. Điều này có nghĩa là message body chứa nhiều khối chunk. Mỗi khối chunk bao gồm:
  - Kích thước khối chunk theo byte, biểu diễn ở dạng hexa (vd 'b' là 11 byte) + kí tự xuống dòng
  - Nội dung của chunk
Đoạn message kết thúc với 1 chunk có size = 0
```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

b
q=smuggling
0
```

Ta hoàn toàn có thể sử dụng cả 2 để tạo xung đột. Mặc dù thông số kỹ thuật của HTTP đã chỉ ra nếu có cả 2 thì CL header sẽ không được dùng. Tuy nhiên nhiều trường hợp như server không hôc trợ TE header, hay server không xử lý nếu TE header bị xáo trộn, thì vẫn gây ra lỗ hổng. 

-> Chốt lại: nếu FE server và BE server có hoạt động khác nhau đối với TE header, thì có thể không đồng nhất về ranh giới giữa 2 request, dẫn đến lỗ hổng smuggling
## How to perform an HTTP request smuggling attack
Request smuggling attacks liên quan đến việc đặt cả `Content-Length` header và `Transfer-Encoding` header vào 1 HTTP request duy nhất và sử dụng chúng để khiến FE server và BE server xử lý request khác nhau
+ CL.TE: FE server sử dụng CL header, BE server sử dụng TE header
+ TE.CL: ngược lại phía trên
+ TE.TE: cả 2 server đều sử dụng TE header, nhưng 1 server có thể không xử lý do header bị ta xáo trộn 
### CL.TE vulnerabilities
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 14
Transfer-Encoding: chunked

0

GET /home
```
Cả đoạn từ sô `0` đến `home` là 14 byte nên FE server gửi cho qua, nhưng phía BE server lại thấy số 0 là chunk size 0 - tức kết thúc 1 chunk, nó sẽ coi là hết request và  đoạn `GET /home` là của request kế
### TE.CL vulnerabilities
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```
FE server sử dụng TE header, và ta có 1 khối chunk hợp lệ tại phần body. Nhưng khi đến BE server, CL header có giá trị = 3, tức là `8\r\n` là hết request, đoạn SMUGGLED đến hết là bắt đầu của request kế
### TE.TE behavior: obfuscating the TE header
Cả 2 server đều có thể sử dụng TE header, nhưng 1 server không xừ lý được vì bị xáo trộn header

![image](https://user-images.githubusercontent.com/97771705/232240895-ade2995c-f2a3-4f19-8b1c-ee0ab53a3675.png)

Cần tìm 1 biến thể sao cho 1 server có thể xử lý nó, trong khi server kia bảo qua nó.
# How to identify HTTP request smuggling vulnerabilities
## Finding HTTP request smuggling vulnerabilities using timing techniques
Cách hiệu quả nhất để phát hiện các lỗ hổng HTTP request smuggling là gửi các request sẽ gây ra sự chậm trễ trong respone của app nếu có lỗ hổng.
### Finding CL.TE vulnerabilities using timing techniques
```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X
```
FE server dùng CL header, nên nó chỉ lấy đủ 4 byte là `1 /n A /n`, còn `X` bị bỏ qua, đến BE server, vì thiếu chunk size 0 để kết thúc nên server sẽ đợi chunk size 0 được chuyển đến và gây ra sự chậm trễ
### Finding TE.CL vulnerabilities using timing techniques
```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```
Tương tự, phía FE chỉ gửi đoạn `0/r/n/r/n`, bỏ qua `X`, trong khi BE server chỉ nhận được 5 byte, đợi thêm 1 byte nữa gây ra sự chậm trễ
## Confirming HTTP request smuggling vulnerabilities using differential responses
1 số lưu ý khi xác nhận HRS bằng các response khác biệt
+ Attack request và nomal request phải được gửi bằng các kết nối mạng khác nhau
+ 2 request phải sử dụng cùng tên URL và parameter càng nhiều càng tốt để tăng khả năng cả 2 đều được xử lý bởi cùng 1 BE server
+ Nếu 2 request được gửi đến 2 BE server thì kiểm tra thất bại, do đó nên gửi nomal request ngay sau attack request 
+ Cẩn trọng vì có thể ngay sau attack request là 1 request từ người dùng ->ảnh hưởng xấu đến người dùng
### Confirming CL.TE vulnerabilities using differential responses
![image](https://user-images.githubusercontent.com/97771705/232264545-5973bc03-c1b4-4357-be9d-5fac8d7f5673.png)

### Confirming TE.CL vulnerabilities using differential responses
![image](https://user-images.githubusercontent.com/97771705/232264603-de0553f8-c62c-40e6-be5d-72ad19218a5d.png)

![image](https://user-images.githubusercontent.com/97771705/232264609-81dc0f75-8818-49f9-8fe5-598a4b23b26a.png)

# How to exploit HTTP request smuggling vulnerabilities
## Using HTTP request smuggling to bypass front-end security controls
Đôi khi FE server có sử dụng sercurity control, và request sẽ chỉ được chuyển đến BE server khi nó đã thông qua sercurity control của FE server.

Lỗ hổng ở đây là BE server hoàn toàn tin tưởng FE server, cho rằng request đã qua FE server thì không cần kiểm tra gì nữa và tiếp tực xử lý.

![image](https://user-images.githubusercontent.com/97771705/232265249-1591a0db-27b2-496a-a983-c3ed6ca9d7c2.png)

## Revealing front-end request rewriting
Một vài trường hợp, FE server sẽ viết lại request trước khi chúng chuyển đến BE server, thường là thêm vài header VD:
+ Có thể thêm X-Forwarded-For header chứa user IP
+ thêm 1 só thông tin nhạy cảm có lợi cho các cuộc tấn công khác

Nếu trong smuggle request thiếu một vài header thường được FE server thêm vào thì BE server có thể k xử lý theo cách thông thường (ta cần BE server xử lý bình thường thì mới biết được là HRS có thành công không)

Cách để biết FE server thêm header nào:
+ Tìm POST request mà nó reflect giá trị của parameter trong response

![image](https://user-images.githubusercontent.com/97771705/232265655-b18d7cd9-6db1-4829-9ab2-18c644026879.png)

+ Chuyển parameter có reflect xuống cuối message body
+ send smuggle request và ngay sau đó send request thường

![image](https://user-images.githubusercontent.com/97771705/232265773-29cb8632-84d4-4196-85af-048bf53eba15.png)

Khi đã biết FE server thêm header gì thì tiếp tục quẩy thôi :)
## Bypassing client authentication
Trong TLS handshake, server tự xác thực với client bằng cách cung cấp chứng chỉ. Chứng chỉ này chứa Common Name (CN), mà phải khớp với hostname đã dăng ký của họ. Client sẽ sử dụng nó để xác minh server có hợp pháp hay k.

Phiên bản nâng cấp của điều này là triển khai xác thực lần nhau, tức là client cũng phải gửi chứng chỉ cho server. Trong trường hợp này Common Name thường là username hoặc cái gì đó tương tự. Và FE server thường thêm header chứa Common Name của client trước khi đến BE server. Đương nhiên BE server cũng tin tưởng tuyệt đối.

Nếu chỉ gửi request đơn thuần có header chứa Common Name thì sẽ bị FE server ghi đè ngay -> Smuggle request là sự lựa chọn hoàn hảo để bypass

![image](https://user-images.githubusercontent.com/97771705/232266172-3a42fbb4-46e4-4002-b62f-ad9ff274154c.png)

![image](https://user-images.githubusercontent.com/97771705/232266191-5fbef9d3-1943-4e43-a8cd-70443c795f93.png)

## Capturing other users' requests
Nếu ứng dụng có chức năng nào đó cho phép lưu trữ và truy xuất văn bản (ví dụ như chức năng comment của lab, email, mô tả hồ sơ...) thì ta hoàn toàn có thể chặn bắt request của user. Có thể dùng điều này để lấy csrf token của user khác. 

Cách làm như phần `Revealing front-end request rewriting`, Chọn chức năng comment của lab, đưa parameter `commment=` xuống cuối và send smuggle request. Phần văn bản in ra sẽ chứa nội dung của request kế.

![image](https://user-images.githubusercontent.com/97771705/232266449-6536a337-03ec-4eea-ac62-3331d672a9da.png)

## Using HTTP request smuggling to exploit reflected XSS
1 app có HRS thì cũng có thể có XSS reflect. Cách tiếp cận nahyf tốt hươn so với XSS thông thường:
+ Không cần user tương tác: gửi 1 phát là request kế của user dính ngay
+ Khai thác XSS tại những chỗ mà XSS thông thường không khai thác được( ví dụ là header user-agent) 

![image](https://user-images.githubusercontent.com/97771705/232266973-91d4bc72-126a-4bd0-b42a-c64b995c2264.png)

## Using HTTP request smuggling to turn an on-site redirect into an open redirect
Nhiều app thực hiện on-site redirect từ 1 URL tới 1 URL khác, thông qua việc lấy hostname từ Host header đặt vào redirect URL 
```
GET /home HTTP/1.1
Host: normal-website.com

HTTP/1.1 301 Moved Permanently
Location: https://normal-website.com/home/
```
![image](https://user-images.githubusercontent.com/97771705/232268188-8b17b379-9451-4574-b786-b98f715c7e75.png)

### Turning root-relative redirects into open redirects

![image](https://user-images.githubusercontent.com/97771705/232268660-ef253e5e-b085-4255-8436-6156f54d9eda.png)

## Using HTTP request smuggling to perform web cache poisoning
## Using HTTP request smuggling to perform web cache deception

## Advanced HTTP request smuggling
## Browser-powered request smuggling
## How to prevent HTTP request smuggling vulnerabilities

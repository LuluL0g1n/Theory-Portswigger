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
## How to identify HTTP request smuggling vulnerabilities

## How to exploit HTTP request smuggling vulnerabilities
## Advanced HTTP request smuggling
## Browser-powered request smuggling
## How to prevent HTTP request smuggling vulnerabilities

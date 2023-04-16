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
Nếu bất kỳ phần nào của cơ sở hạ tầng frontend lưu trữ nội dung vào bộ nhớ cache thì có thể poison bộ nhớ cache với off-site redirect response.

![image](https://user-images.githubusercontent.com/97771705/232284594-d1315282-9d63-41fe-971f-80ded6e93f9c.png)


## Using HTTP request smuggling to perform web cache deception
Sự khác biệt giwuax web cache poisoning và web cache deception:
+ Attacker khiến app lưu trữ malicious content ở cache, và content đó tiếp tục được cache cung cấp cho user khác
+ Attacker khiến app lưu trữ content nhạy cảm của user khác vào cache, sau đó truy xuất những content này 

![image](https://user-images.githubusercontent.com/97771705/232286490-707c3820-dc4e-47ff-9e39-3a9767a46e95.png)

# Advanced HTTP request smuggling
## HTTP/2 request smuggling
### HTTP/2 message length
Request smuggling về cơ bản là khai thác sự khác biệt giữa cách các server khác nhau diễn giải độ dài của request.

HTTP/2 message được gửi dưới dạng một loạt các "frame" riêng biệt. Trước mỗi frame là 1 trường độ dài, cho phép server biết chính xác bao nhiêu byte cần đọc.

-> lenght của request là tổng lenght của các frame

Nếu trang web sử dụng HTTP/2 từ đầu đến cuối thì không thể gây ra sự mơ hồ về cách xác định lenght của các server, nhưng trong thực tế, việc hạ cấp HTTP/2 là phổ biến  
### HTTP/2 downgrading
Hạ cấp HTTP/2 là quá trình viết lại các HTTP/2 request bằng cách sử dụng cú pháp HTTP/1 để tạo ra một HTTP/1 request tương đương. FE server thường làm điều này để có thể giao tiếp vs BE server chỉ hỗ trợ HTTP/1. Khi BE server trả lại respone thì FE server lại viết lại HTTP/1 thành HTTP/2 response cho ứng dụng.

![image](https://user-images.githubusercontent.com/97771705/232287117-ddd9b85c-97fe-4fbc-9c28-499b2f7967b5.png)

![image](https://user-images.githubusercontent.com/97771705/232287466-b8c5a151-754c-4feb-b268-aaf2583d2a43.png)

### H2.CL vulnerabilities
HTTP/2 request không cần phải chỉ định rõ ràng lenght của nó trong header. Tức là khi downgrading, FE server tự thêm CL header dựa vào cách tính tổng các trường độ dài của mỗi frame để cho ra lenght của request.

Nhưng HTTP/2 cũng có thể tự thêm CL header. Vậy điều gì sẽ xảy ra nếu CL header không khớp với giá trị tính toán tổng các trường độ dài của frame?

Khi đó, tại FE server, bắt buộc phần kết thúc request phải dựa vào tổng lenght của frame, nhưng khi Downgrading thì lại gây lú cho BE server bẳng CL header -> request ngắn hơn, phân còn lại được coi là request mới

![image](https://user-images.githubusercontent.com/97771705/232288007-1af76a9b-e1c5-4d30-ac54-b05a1ca4b51c.png)

### H2.TE vulnerabilities
tương tự lỗ hổng trên, nhưng ta tự thêm TE header 

![image](https://user-images.githubusercontent.com/97771705/232288150-be073d64-55d1-4846-8f92-b8682a037976.png)

### Hidden HTTP/2 support
## Response queue poisoning
Queue poisoning là attack mà khiến FE server gửi response cho request khác với request đã yêu cầu response đó.

Ví dụ cụ thể là gửi 1 smuggle request, phía FE cho rằng tất cả là 1 request, nhưng phía BE xác nhận là 2 request, do đó gửi 2 response trả về (R1 và R2). Khi trả về FE, nó chỉ nhận R1 cho smuggle request, và R2 sẽ phải ở trong queue. Nếu ngay sau smuggle request có normal request từ user, thì R2 sẽ được gửi cho user và response của user sẽ tiếp tục để trong Queue...

![image](https://user-images.githubusercontent.com/97771705/232290795-59f79f5a-c27e-4642-8d6d-9db688674852.png)

Các điều kiện để có thể sử dụng attack này:
+ TCP connect giữa FE và BE phải được tái sử dụng cho nhiều request response
+ Smuggle request phải được send thành công, và mỗi request(thường smuggle request tại BE server được tách ra làm 2 request do sự mơ hồ về điểm kết thúc của request) đều được BE server trả về response độc lập
+ Attack không làm tắt connect TCP

Có vài trường hợp cần mắc khi sử dụng attack này:
+ Nếu phần smuggle chỉ chứa 1 vài header, thì khi request tiếp theo gắn vào, BE sẽ chỉ coi là 2 request, mà ta cần BE hiểu là 3 request trở lên
+ Nếu phần smuugle có chứa đầy đủ header và 1 phần body, rất khó để xác định chính xác CL header sẽ cắt đến phần nào của user request, dẫn đến request thứ 3 chỉ là vài byte riêng lẻ -> server gặp trường hợp như này là cắt cmn TCP connect  

![image](https://user-images.githubusercontent.com/97771705/232290464-42dad11f-4997-42b2-a2b5-b4f3c2731e13.png)

Ví dụ về 1 attack thành công

![image](https://user-images.githubusercontent.com/97771705/232290762-1fca8868-4c7f-4ec8-953b-73417305e2ef.png)

Trường hợp trên chỉ dừng ở request thứ 3, tức là chỉ có thể chuyển respone sai đến user. Nếu attacker gửi request thứ 4 1 cách bình thường ngay sau đó, họ có thể đọc được thông tin trong response của user

![image](https://user-images.githubusercontent.com/97771705/232290973-20921f70-73a1-48ff-9683-38d4bf29da34.png)

## Request smuggling via CRLF injection
Các trang web có thể chặn H2.CL hoặc H2.TE basic attack bằng vài cách như xác thực CL header hoặc xóa TE header trước khi downgrading.

Tại HTTP/1, dấu `/n` có thể gây lú khi FE cho rằng nó chỉ là 1 kí tự, nhưng BE lại cho là kí tự xuống dòng. Tuy nhiên điều này không xảy ra với `/r/n` vì cả 2 server đều đồng ý đây là chấm dứt của header
```
Foo: bar\nTransfer-Encoding: chunked
```
HTTP/2 thì không như thế, khi mà thứ duy nhất xác định header end hay không là trường độ dài của mỗi frame -> khi downgrading thì HTTP/1 sẽ coi `/r/n` là end của 1 header 

![image](https://user-images.githubusercontent.com/97771705/232291604-0d3f031f-b14d-48db-9c92-5e0742136ff5.png)

=>
```
Foo: bar
Transfer-Encoding: chunked
```
## HTTP/2 request splitting
Trong trường hợp CL header bị loại bỏ trước khi downgrading và BE không hỗ trợ chunked encoding thì HTTP/2 request spliting là sự lựa chọn hoàn hảo để bypass

![image](https://user-images.githubusercontent.com/97771705/232292001-aee5e9ec-353f-47b7-b35d-264e0bce7f90.png)

### Accounting for front-end rewriting
Nhiều khi FE server bắt buộc thêm 1 header trước khi downgrading và send tới BE. 

Chẳng hạn FE thêm `Host` header vào cuối list header và ta sử dụng smuggle request như phần HTTP/2 request splitting

![image](https://user-images.githubusercontent.com/97771705/232293369-0723539e-0469-4553-97b7-742e282ccad4.png)

Như này, FE sẽ thêm vào sau header `foo`. Thêm xong quá trình downgrading diễn ra và request 1 sẽ không có Host header, trong khi request 2 có 2 header

Giải pháp

![image](https://user-images.githubusercontent.com/97771705/232293727-08d928d2-a4b0-463a-88d4-c4d34acf3b16.png)

## HTTP request tunnelling
Tất cả những phần trên là chỉ sử dụng 1 kết nối giữa FE và BE cho nhiều request response. 

Nhiều server chỉ cho phép những request thuộc cung 1 IP hoặccùng 1 client sử dụng lại kết nối.

Khi đó, ngay cả việc sử dụng queue poisoning thì response thứ 2 sẽ bị ẩn hoàn toàn khỏi FE do ta không lấy được request của user khác (do khác connect)

![image](https://user-images.githubusercontent.com/97771705/232296305-409f8443-9737-480d-81fd-184583eceecb.png)

![image](https://user-images.githubusercontent.com/97771705/232296334-ce0327e8-28ce-46de-897c-4d8efedf269f.png)

### Request tunnelling with HTTP/2
< khó quá để sau> 
## Browser-powered request smuggling
### CL.0 request smuggling
Đôi khi BE bị thuyết phục bỏ qua CL header, tức là mọi request đều kết thúc tại header và phần body coi như bỏ. Hay nói cách khác là `Content-Lenght: 0`. 

Phần body được coi là 1 request mới -> khai thác tiếp :)

Lỗ hổng H2.0 tương tự, khi mà BE bị thuyết phục bỏ qua CL header sau khi downgrading
### Client-side desync attacks <để sau>
### Pause-based desync attacks <để sau>
## How to prevent HTTP request smuggling vulnerabilities
+ Sử dụng HTTP/2 end to end và tắt tính năng hạ cấp HTTP. Nếu không thể tránh được việc sử dụng downgrading, xác thưc request trước khi nó downgrading: từ chối các request có kí tự new line(/r/n) trong header, dấu `:` trong header name hay dấu `space` trong request method
+ Làm cho FE server bình thường hóa các request mơ hồ và BE server từ chối bất kỳ request nào vẫn còn mơ hồ. Sau đó ngắt kết nối TCP.
+ Luôn cho rằng các request sẽ có phần body để chống CL.0
+ 

# jwt_springboot

About JWT - how JWT works in spring_boot

+ Bắt đầu với file WebSecurityConfig. Nơi mình sẽ khai báo các webservice nào bắt buộc phải login và cái nào không. Đồng thời
mình nhúng JwtAuthenticationEntryPoint và jwtRequestFilter vào sử dụng

+ Mục đích của JwtAuthenticationEntryPoint là để reject tất cả các request mà không authenticate vào hệ thống. Nếu 1 request mà không đăng nhập thì sẽ ném về lỗi 403

+ Mục đích của jwtRequestFilter kiểm tra tất cả các request truyền lên server có hợp lệ hay không bằng việc kiểm tra token trên header
JwtTokenUtil có nhiệm vụ sinh ra token dự vào key mình cung cấp trong file application.properties


## JSON WEB TOKEN 
1. Vì sao chúng ta cần Jwt ?<br>
- Lấy ví dụ về mua hàng online trên mạng và thanh toán tiền qua thẻ visa hoặc master. Khi các em thanh toán qua một ứng dụng trung gian để mua sản phẩm thì ứng dụng trung gian sẽ yêu cầu các em nhập vào số thẻ và mã cvv. Như các em thấy nếu thông tin này không bị mã hoá trước khi truyền đi trên mạng. Hacker hoàn toàn có thể lấy được thông tin về số thẻ và mã cvv của mình. Nếu có được 2 thông tin này hacker hoàn toàn có thể lấy hết số tiền trong thẻ hoặc sẽ dùng thông tin đó đi mua hàng .Như vậy rất nguy hiểm khi thông tin chúng ta truyền trên mạng mà không được bảo mật hay nói cách khác là thông tin chúng ta không được mã hoá an toàn sẽ dẫn đến rất nhiều hệ luỵ sau này.
Hoặc chúng ta sử dụng session và cookie để lưu thông tin cho những ứng dụng web. Nhưng nếu ứng dụng của ta là mobile thì không có session và cookie. Mà thay vào đó là dùng JWT

2. JSON Web Token là gì ?<br>
- JSON Web Token (JWT) là 1 tiêu chuẩn mở (RFC 7519) định nghĩa cách thức truyền tin an toàn giữa client và server bằng đối tượng JSON. Thông tin này có thể được xác thực và đánh dấu tin cậy nhờ vào “chữ ký” của nó. Phần chữ ký của JWT sẽ được mã hóa nhằm tránh các hacker có thể lấy nó thông qua mạng. Bằng các thuật toán mã hoá có tên gọi là HMAC hoặc các thuật toán mã hoá dữ liệu là RSA thì dữ liệu chúng ta sẽ được mã hoá trên mạng, hacker có lấy được cũng khó mà giải mã ra các dữ liệu. Như vậy mình sẽ yên tâm hơn vì các dữ liệu mình truyền đi đều đã được mã hoá. 

3. Cấu tạo của 1 JWT<br>
JWT bao gồm 3 phần và ngăn cách nhau bởi dấu chấm :
   - Header
   - Payload
   - Signature (chữ ký số )

Ví dụ sau đây là một token được sinh ra : <br>
  ###  eyJhbGciOiJIUzUxMiJ9 . eyJzdWIiOiJuZ3V5ZW4iLCJleHAiOjE1Njg3NTAxMTEsImlhdCI6MTU2ODczMjExMX0 . mPuurljzpycuyy0d_B0GNVPBz7SEpPCPIoGGy2lUVgJ9rLlRJkDCdG2vwkXITUsJ4dnU5IF178yXv34izGPcpw

Header là    : eyJhbGciOiJIUzUxMiJ9
Payload là   : eyJzdWIiOiJuZ3V5ZW4iLCJleHAiOjE1Njg3NTAxMTEsImlhdCI6MTU2ODczMjExMX0
Chữ ký số là : mPuurljzpycuyy0d_B0GNVPBz7SEpPCPIoGGy2lUVgJ9rLlRJkDCdG2vwkXITUsJ4dnU5IF178yXv34izGPcpw

 - Header lưu trữ thông tin về loại token và thuật toán mã hoá đang dùng Header bao gồm hai phần chính: loại token (mặc định là JWT - Thông tin này cho biết đây là một Token JWT) và thuật toán đã dùng để mã hóa (HMAC SHA256 - HS256 hoặc RSA). { “alg”: “HS256”, “typ”: “JWT” }

 - Payload là phần sẽ chứa nội dung server (dữ liệu) ta truyền lên server. Ngoài ra nó còn chứa đựng các thông tin về tolkien như ngày hết hạn, ngày sinh ra token,subject, etc.
Payload chứa các claims. Claims là một các biểu thức về một thực thể (chẳng hạn user) và một số metadata phụ trợ. metadata là bắt buộc, số còn lại nên tuân theo để JWT hợp lệ và đầy đủ thông tin: iss (issuer), iat (issued-at time) exp (expiration time), sub (subject), aud (audience), jti (Unique Identifier cho JWT, Can be used to prevent the JWT from being replayed. This is helpful for a one time use token

 - Signature là một chuỗi được mã hoá bao gồm các thông tin header + payload + chữ ký Chữ ký Signature trong JWT là một chuỗi được mã hóa bởi header, payload cùng với một chuỗi bí mật theo nguyên tắc sau:
HMACSHA256( base64UrlEncode(header) + “.” + base64UrlEncode(payload), secret)

 - Trong đó từ secret là cái đặc biệt quan trọng, chỉ một số người được biết giá trị này. Nó được kết hợp với các thuật toán để cho ra một token bảo mật. Ví dụ trong trường hợp xấu nhất header và payload bị lỗi thì secret chính là giá trị không phải ai cũng biết mà giải mã. Nó giúp cho token được tạo ra thêm một lớp bảo mật. Tăng tính an toàn cho token. Chính vì đặc điểm này nên token được sử dụng nhiều trong các ứng dụng.

4. Cơ chế hoạt động của token

 - Khi người dùng đăng nhập vào hệ thống. Trong trường hợp này anh đang chọn là người dùng nhập đúng username và password. Khi server đã kiểm tra là ok thì lúc đó server sẽ tạo ra một dãy token ví dụ như eyJhbGciOiJIUzUxMiJ9.  eyJzdWIiOiJuZ3V5ZW4iLCJleHAiOjE1Njg3NTAxMTEsImlhdCI6MTU2ODczMjExMX0.mPuurljzpycuyy0d_B0GNVPBz7SEpPCPIoGGy2lUVgJ9rLlRJkDCdG2vwkXITUsJ4dnU5IF178yXv34izGPcpw

Dãy token này là duy nhất và không bị trùng lặp. Đồng thời không thể dịch ngược lại được các thông tin mà đã mã hoá. Sau đó server sẽ gửi trả token này về lại cho client.

Tất các các hành động khác của client sau khi login thành công. Thì khi gọi một hành động nào trên server thì client sẽ gửi kèm thêm một thông tin token nữa lên server.

Ở phía server sau khi nhận được token gửi lên từ client thì server sẽ kiểm tra xem token đó có hợp lệ hay không. Nếu hacker hoặc một ai đó sửa chuỗi token ở trên thì server sẽ nhận biết được là token đó không hợp lệ và không thực hiện.

Hoặc sẽ có những trường hợp token hết hạn thì lúc đó client phải bắt buộc đăng nhập lại để nhận lại token mới từ server.

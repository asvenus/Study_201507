# Study_201507
1. Lời nói đầu
   Hiện nay có rất nhiều ứng dụng chat video trực tuyến để làm viêc, chat với bạn bè... như hangout, skype..
Các ứng dụng có thể chạy trên các nền tảng web browser, ios, android... Ta có thể dễ dàng tạo một streaming để chat
video với opentok. Trên browser nó dựa vào nền tảng webRTC để thực hiện việc truyền các gòi tin đa phương tiện bằng 
javascript

2. OpenTok là gì?:
  Opentok là một ứng dụng chat dựa trên nền tảng web. Bạn có thể vào trang chủ của TokBox để tạo một tài khoản đăng nhập và mời mọi người tham gia chat video với bạn thông qua trình duyệt web. Một tính năng cao cấp khác của TokBox là khả năng tích hợp với các dịch vụ web khác như Facebook, Meebo. Bạn có thể cài đặt add-on TokBox cho Facebook Firefox để có thể sử dụng tính năng chat video nhanh chóng với bạn bè trong tài khoản Facebook.
Opentok sẽ dựa vào nền tảng webRTC để gửi các gói tin đa phương tiện qua javascript giúp tạo một streaming để chat.
  #Có thể tìm hiểu qua opentok sdk cho mobile app
video.
3. Sử dụng OpenTok để tạo 1 kênh chat với Rails
  Ta sẽ sử dụng opentok sdk để tạo một room chat chung. Với rail ta có sẵn gem "opentok".
  Ý tưởng : Sẽ tạo ra các room chat. Mỗi room sẽ có 1 session_id gắn với opentok. Khi user join vào room đó sẽ gọi đến sdk của opentok và nó sẽ tạo ra 1 stream mới .
Đầu tiên chúng ta phải đăng kí 1 app trên trang developercủa opentok: https://tokbox.com/developer/.
Một app trial dùng dc trong 30 ngày, giá để mua licence là 50$/month(đắt quá).
Một app cũng giống như facebook app sẽ có cặp api_key và secrets.
Trên serer side:
- Khi tạo 1 room mới ta sẽ taọ 1 session_id của room. Session này dc tạo ra từ sdk opentok. Nó cho phép người dùng
access đến các fucntion của opentok của ta. Table room có attributes: {id, session_id}
- Khi 1 người dùng join vào room, Opentok sẽ tạo ra 1 session_id của người dùng thông qua session_id của room 
đó. 
Phía dưới client side.
Opentok có hẳn 1 thư viện js để giúp hiển thị các màn hình video chat của từng client. có thể tải tại đây:
http://static.opentok.com/webrtc/v2.0/js/TB.min.js"
Phía dưới client sẽ tạo ra 1 session thông qua session_id của room. Session sẽ lắng nghe nếu có 1 client mới join vào sẽ tạo ra 1 stream mới và add vào thẻ body thông qua 2 event handles:
  sessionConnected: đễ publissh một client mới sau khi stream dc create
  streamCreated: Sẽ create một stream mới và gán nó vào element cần hiển thị trên view.
Cuối cùng là connect với open tok thông qua api_key và token của client đó: 
session.connect(apiKey, token);


4. Demo


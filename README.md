# Study_201507
## I/Lời nói đầu
   Hiện nay có rất nhiều ứng dụng chat video trực tuyến để làm viêc, chat với bạn bè... như hangout, skype..
Các ứng dụng có thể chạy trên các nền tảng web browser, ios, android... Ta có thể dễ dàng tạo một streaming để chat
video với opentok. Trên browser nó dựa vào nền tảng webRTC để thực hiện việc truyền các gòi tin đa phương tiện bằng 
javascript

## II/ OpenTok là gì?:
  Opentok là một ứng dụng chat dựa trên nền tảng web. Bạn có thể vào trang chủ của TokBox để tạo một tài khoản đăng nhập và mời mọi người tham gia chat video với bạn thông qua trình duyệt web. Một tính năng cao cấp khác của TokBox là khả năng tích hợp với các dịch vụ web khác như Facebook, Meebo. Bạn có thể cài đặt add-on TokBox cho Facebook Firefox để có thể sử dụng tính năng chat video nhanh chóng với bạn bè trong tài khoản Facebook.
Opentok sẽ dựa vào nền tảng webRTC để gửi các gói tin đa phương tiện qua javascript giúp tạo một streaming để chat.
TODO: Có thể tìm hiểu qua opentok sdk cho mobile app
video.
## III/ Sử dụng OpenTok để tạo 1 kênh chat với Rails
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


## IV/. Demo

Đầu tiên cần 1 table room:
```ruby
class CreateRooms < ActiveRecord::Migration
  def change
    create_table :rooms do |t|
      t.string :session_id
      t.timestamps null: false
    end
  end
end
```
Trên server side:
Trong rooms_controlle.rb sẽ có các action  `create` và  `show`.
Ta sẽ tạo một room  và gắn session_id dc lấy từ opentok.
```ruby 
def create
@opentok = OpenTok::OpenTok.new API_KEY, API_SECRET
    session = @opentok.create_session
    @room = Room.new(session_obj: session.session_id)
    .........................
end
```
Trong function trên một @opentok được tạo ra từ app ta đăng kí với key and secret.
1 session dạng object: 
``` ruby
=> #<OpenTok::Session:0x007fe35167e580
 @api_key="#{api_key}",
 @api_secret="#{api_secret}",
 @archive_mode=:manual,
 @location=nil,
 @media_mode=:relayed,
 @session_id="2_MX40NTI5MDg2Mn5-MTQzNzcxNTc5MjE4NX5wSmRyZzc4cEVRMlBMOUp4bnBRMGNwV2h-UH4">
```
Ta lưu session_id của room vào bảng room. 1 session là duy nhất.

Khi 1 user join vào room sẽ gọi đến action `show` Tại đây ta sẽ truyền 2 thông số xuống view bao gồm @session_id
của room và token của client đó.
```ruby
    @session_id = @room.session_obj
    @token = OpenTok::OpenTok.new(api_key,api_secret).generate_token @session_id
```
Dưới client side:
  Bên dưới client sẽ nhận được 3 biến truyền xuống( api_key, session_id và token)
chúng ta khởi tạo 1 session bằng hàm 
```javascript
var session = TB.initSession("<%= @room.session_id %>")
```
Một session sẽ cung cấp các function của open tok. Các functionaly có thể tham khảo ơ đây: https://tokbox.com/developer/sdks/js/reference/Session.html.
2 event hander đáng chú ý của nó là sessionConnected và streamCreated hiểu nôm na sẽ tạo ra 1 stream mới khi client join room và connect tới room đó sau khi stream đó dc tạo.

```javascript
    session.addEventListener("sessionConnected", sessionConnectedHandler);
      session.addEventListener("streamCreated", streamCreatedHandler);
      session.connect(api_key, "<%= @tok_token %>");
      function sessionConnectedHandler(event) {
         subscribeToStreams(event.streams);
         session.publish();
      }

      function streamCreatedHandler(event) {
        subscribeToStreams(event.streams);
      }

      function subscribeToStreams(streams) {
        for (var i = 0; i < streams.length; i++) {
        // Make sure we don't subscribe to ourself
          if (streams[i].connection.connectionId == session.connection.connectionId) {
            return;
          }

          // Create the div to put the subscriber element in to
          var div = document.createElement('div');
          div.setAttribute('id', 'stream' + streams[i].streamId);
          document.body.appendChild(div);

          // Subscribe to the stream
          session.subscribe(streams[i], div.id);
        }
   ```
## V/ Tài liệu tham khảo:
https://github.com/loganathan-s/vide0-chat-using-tokbox
https://github.com/opentok/Opentok-Ruby-SDK
http://www.tokbox.com/blog/building-a-video-party-app-with-ruby-on-rails/
https://tokbox.com/developer/



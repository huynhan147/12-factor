
## IV. Dịch vụ sao lưu

### Coi dịch vụ sao lưu như tài nguyên đính kèm

Một _dịch vụ sao lưu_ là bất kỳ dịch vụ nào mà ứng dụng sử dụng kết nối mạng như là một phần trong quá trình họat động bình thường của nó . Các ví dụ bao gồm kho dữ liệu (như [MySQL](1) hoặc [CouchDB](2)), các hệ thống nhắn tin/xếp hàng (như [RabbitMQ](3) or [Beanstalkd](4)), các dịch vụ STMP cho gửi mail (such as [Postfix](5)), và các hệ thống caching (như [Memcached][6]).

Các dịch vụ sao lưu như cơ sở dữ liệu thường được quản lý bới các quản trị viên hệ thống như là các triển khai trong quá trình chạy ứng dụng. Ngoài các dịch vụ được quản lý cục bộ này, ứng dụng cũng có thể có các dịch vụ được cung cấp và quản lý bới bên thứ 3. Ví dụ bao gôm dịch vụ STMP (như [Postmark](7)), dịch vụ thu thập số liệu (như [New Relic](8) hoặc (Loggly)(9)), dịch vụ binary asset (như  [Amazon S3](10)), và thậm chí cả dịch vụ sử dụng API-accessible (như [Twitter](11), [Google Maps](12), or [Last.fm](13)).

**Code ứng dụng theo 12-chuẩn không phân biệt dịch vụ cục bộ hay dịch vụ bên thứ ba.** Đối với ứng dụng,cả hai đều là tài nguyên đính kèm, được truy cập qua URL hoặc các định vị/thông tin xác thực khác được lưu trong [cấu hình](14). Một bản [triển khai](15) của ứng dụng 12-chuẩn có thể hoán đổi một cơ sở dữ liệu MySQL cục bộ với một cơ sở dữ liệu được quản lý bởi bên thứ ba (như [Amazon RDS](16)) mà không có bất kỳ thay đổi nào đối với code của ứng dụng. Tương tự như vậy, một máy chủ STMP cục bộ có thể hoán đổi với dịch vụ STMP của bên thứ ba (như Postmark) mà không phải thay đổi code. Trong cả hai trường hợp, chỉ tài nguyên xử lý trong cấu hình cần được thay đổi.

Mỗi dịch vụ sao lưu riêng biệt là một _tài nguyên_. Ví dụ, một cơ sở dữ liệu MySQL là một tài nguyên; hai cơ sở dữ liệu MySQL (được sử dụng để sharding ở tầng ứng dụng) đủ điều kiện là hai tài nguyên riêng biệt. Ứng dụng 12-chuẩn coi các cơ sở dữ liệu này như  _các tài nguyên đính kèm_, cho biết kết nối lỏng lẻo của chúng với triển khai mà chúng được gắn vào.


Tài nguyên có thể được gắn vào và tách ra khỏi các triển khai theo ý muốn. Ví dụ: nếu cơ sở dữ liệu của ứng dụng bị lỗi do sự cố phần cứng, người quản trị ứng dụng có thể chuyển qua một máy chủ cơ sở dữ liệu mới được khôi phục từ bản sao lưu gần đây. Cơ sở dữ liệu production hiện tại có thể được tách ra và cơ sở dữ liệu mới được thêm vào - tất cả đều không có bất kỳ thay đổi code nào.
## V. Xây dựng, phát hành, chạy

### Tách biệt giai đoạn xây dựng và chạy

Một [codebase](1) được chuyển thành một bản triển khai (không phát triển) qua ba giai đoạn:

* _Giai đoạn xây dựng_ là một quá trình chuyển đổi một code repo thành một gói thực thi được gọi là một _build_ . Sử dụng phiên bản của code tại một commit được chỉ định bởi quá trình triển khai, giai đoạn xây dựng sẽ lấy các [phụ thuộc] (2) vendors và biên dịch các file nhị phân và asset.
* _Giai đoạn phát hành_ lấy bản build được tạo bởi giai đoạn xây dựng và kết hợp nó với [cấu hình](3) hiện tại của bản triển khai. Kết quả bản _phát hành_ chửa cả bản build và cấu hình và sẵn sàng để thực thi ngay trong môi trường thực thi.
* _Giai đoạn chạy_ (còn được gọi là "thời gian chạy") chạy ứng dụng trong môi trường thực thi, bằng cách khởi chạy một số tập các [tiến trình](4) của ứng dụng đối với bản phát hành đã chọn.

**Ứng dụng 12-chuẩn sử dụng tách biệt rõ ràng giữa giai đoạn xây dựng, phát hành và chạy.** Ví dụ, không thể thay đổi code trong thời gian chạy, vì không có cách nào để truyền các thay đổi đó trở lại giai đoạn xây dựng

Các công cụ triển khai thường sẽ chấp nhận các công cụ quản lý xuất bản, đáng chủ ý nhất là có thể roll back về xuất bản trước đó. Ví dụ, công cụ triển khai [Capistrano] (6) lưu trữ các bản phát hành trong một thư mục con có tên `release`, trong đó bản phát hành hiện tại là một liên kết tượng trưng đến thư mục phát hành hiện tại. Lệnh `rollback` của nó giúp bạn dễ dàng roll back về bản phát hành trước.

Mỗi bản phát hành nên luôn có một ID duy nhất, như mốc thời gian của bản phát hành (như là `2011-04-06-20:32:17`) hoặc số tự  tăng (như `v100`). Các bản phát hành chỉ được thêm vào hồ sơ và một bản phát hảnh không thể thay đổi khi nó được tạo ra.Mọi thay đổi phải tạo bản phát hành mới.

Các bản build sẽ được khởi tạo bởi nhà phát triển ứng dụng mỗi khi code mới được triển khai. Ngược lại, thời gian thực thi chạy có thể tự động xảy ra trong một số trường hợp như khi server khởi động lại, hoặc một tiến trình crash được khởi động lại bởi trình quản lý tiến trình. Vì thế, giai đoạn chạy nên được giữ càng ít phần chuyển tiếp càng tốt, vì các vấn đề ngăn ứng dụng chạy có thể làm cho ứng dụng ngừng hoạt động vào giữa đêm khi không có nhà phát triển nào có mặt. Giai đoạn xây dựng có thể phức tạp hơn, vì các lỗi luôn có thể xảy ra đối với một nhà phát triển đang thự hiện triển khai.
## VI. Các tiến trình

### Thực thi ứng dụng dưới dạng một hoặc nhiều tiến trình không trạng thái

Ứng dụng được thực thi trong môi trường thực thi dưới dạng một hoặc nhiều _tiến trình_.

Trong trường hợp đơn giản nhất, code là một script độc lập, môi trường thực thi là laptop cục bộ của developer với ngôn ngữ thời gian chạy đã được cài đặt, và tiến trình được thực hiện thông qua dòng lệnh (ví dụ, `python my_script.py`). Mặt khác, một triển khai production của một ứng dụng phức tạp có thể sử nhiều [loại tiến trình, khởi tạo từ 0 hoặc nhiều tiến trình đang chạy](1).

**Tiến trình 12-chuẩn là không trạng thái và [không chia sẻ][2].** Bất kỳ dữ liệu nào cần lưu trữ đều phải được lưu trong một [dịch vụ sao lưu](3) có trạng thái, thường là cơ sở dữ liệu.

Bộ nhớ hoặc filesystem của tiến trình có thể được sử dụng như là một cache nhỏ gọn, đơn chiều. Ví dụ, tải xuống một tệp lớn, thao tác trên đó và lưu trữ kết quả sau khi thao tác trong cơ sở dữ liệu. Ứng dụng 12-chuẩn không bao giờ giả định mọi thứ được cache trong bộ nhớ hoặc trên đĩa sẽ khả dụng cho một yêu cầu hoặc công việc trong tương lai –  với nhiều tiến trình của mỗi loại đang chạy, khả năng cao là một yêu cầu trong tương lai sẽ được phục vụ bởi một tiến trình khác. Ngay cả khi chỉ chạy một tiến trình trình, một restart (kích hoạt bởi triển khai của code hoặc thay đổi của cấu hình hay môi trường thực thi chuyển tiến trình sang một vị trí vật lí khác) hường sẽ xóa sạch tất cả trạng thái cục bộ (ví dụ :bộ nhớ và filesystem).

Các đóng gói asset như [django-assetpackager](4) sử dụng filesystem như là bộ nhớ đệm cho các asset được biên dịch. Ứng dụng 12- chuẩn thích thực hiện việc biên dịch này trong [giai đoạn xây dựng](5).Các trình đóng gói asset như [Jammit](6) và [Rails asset pipeline](7)  có thể được cấu hình thành các asset package trong giai đoạn xây dựng.

Một số hệ thống web dựa vào ["sticky sessions"](8) – có nghĩa là, lưu dữ liệu phiên người dùng trong bộ nhớ của tiến trình của ứng dụng và đợi các yêu cầu trong tương lai từ cùng một người sẽ được chuyển đến cùng một tiến trình. Sticky sessions ;à vi phạm 12 chuẩn và không bao giờ nên sử dụng hoặc phụ thuộc vào nó.  Các dữ liệu trang thái phiên là một gỉai pháp thay thế tốt cho các kho dữ liệu mà trong hệ thống có quy định giới hạn về thời gian, như [Memcached](9) hoặc [Redis](10).
## VII. Port binding

### Xuất các dịch vụ qua port binding

Các ứng dụng web đôi khi được thực thi bên trong webserver container. Ví dụ, các ứng dụng php có thể chạy như là một module bên trong [Apache HTTPD](1), hoặc các ứng dụng Java có thể chạy bên trong [Tomcat](2).

**Ứng dụng 12- chuẩn hoàn toàn độc lập** và phụ thuộc vào thời gian chạy của máy chủ web vào môi trường thực thi để tạo dịch vụ web-facing. Ứng dụng web **cung cấp HTTP dưới dạng dịch vụ bằng cách liên kết với cổng**, và lắng nghe các yêu cầu đến trên cổng đó.

Trong môi trường phát triển cục bộ, các developer truy cập vào một URL dạng `http://localhost:5000/` để sử dụng các dịch vụ cung cấp bởi ứng dụng của họ. Trong triển khai, tầng điều hướng xử lý các yêu cầu điều hướng từ một tên miền public-facing đến các tiến trình web được gắn liền với cổng.

Điều này thường được thực hiện bằng cách sử dụng [khai báo phụ thuộc](3) để thêm một thư viện webserver vào ứng dụng, như [Tornado](4) cho Python, [Thin](5) cho Ruby, hoặc [Jetty](6) cho Java và các ngôn ngữ chuẩn JVM khác. Điều này xảy ra hoàn toàn trong _không gian người dùng_, có nghĩa là, trong code của ứng dụng. Các quy ước với môi trương thực thi được kết nối tới một cổng để xử lý các yêu cầu.

HTTP không phải là dịch vụ duy nhất có thể được xuất bởi port binding. Gần như bất kỳ loại phần mềm máy chủ nào cũng có thể được chạy thông qua một quá trình gắn kết với một cổng và chờ các yêu cầu gửi đến. Các ví dụ bao gồm [ejabberd](7) ( [XMPP](8)), và [Redis](9) ( [ giao thức Redis][10]).

Cũng lưu ý rằng cách tiếp cận cổng ràng buộc có nghĩa là một ứng dụng có thể trở thành [dịch vụ sao lưu](11) cho một ứng dụng khác, bằng cách cung cấp URL cho ứng dụng nền như một tài nguyên để xử lý trong [cấu hình](12) cho ứng dụng sử dụng.
## VIII. Xử lý đồng thời

### Mở rộng thông qua mô hình tiến trình
Bất kỳ chương trình máy tính nào, khi đã chạy, được đại diện bởi một hoặc nhiều hơn các tiến trình.Các ứng dụng web có rất nhiều hình thức thực thi tiến tình. Ví dụ, các tiến trình PHP chạy như các tiến trình con của Apache, được khởi tạo theo yêu cầu tầm quan trọng theo số lượng của lượng request. Các tiến trình Java lại có một cách tiếp cận đối ngược, với JVM cung cấp một uberprocess lớn lưu trữ khối tài nguyên hệ thống (CPU và bộ nhớ) lúc khởi động, với xử lý đồng thời được quản lý nội bộ thông qua các luồng. Trong cả 2 trường hợp, các tiến trình chạy chỉ hiển thị một cách tối giản tới các người lập trình của ứng dụng.

**rong ứng dụng 12-chuẩn, các tiến trình là tập các lớp citizen đầu tiên.** Các tiến trình trong ứng dụng theo 12 chuẩn tuân theo [mô hình tiến trình unix để chạy các dịch vụ nền](2). Sử dụng mô hình này, developer có thể thiết kế ứng dụng của họ để xử lý các khối lượng công việc khác nhau bằng cách giao mỗi loại công việc cho một _loại tiến trình_.  Ví dụ các yêu cầu HTTP có thể được xử lý bằng một tiến trình web, và các công việc nền tốn nhiều thời gian được xử lý bởi các tiến trình worker.

Điều này không loại trừ các tiến trình độc lập khỏi việc xử lý các kênh ghép nội bộ của chúng, thông qua các luồng bên trong thời gian chạy của VM, hoặc mô hình bất đồng bộ/sự kiện được tìm thấy trong các công cụ như là [EventMachine](3), [Twisted](4), hoặc [Node.js](5). Nhưng một VM độc lập có thể mở rộng rất lớn (mở rộng theo chiều dọc), do vậy ứng dụng cũng phải có khả năng cung cấp nhiều tiến trình chạy trên nhiều máy vật lý.

Mô hình tiến trình thực sự hiệu quả khi đến khi mở rộng.  [không chia sẻ, có thể phân chia theo chiều ngang của các tiến tình theo 12 - chuẩn](6)  có nghĩa là thêm nhiều xử lý đồng thời là một hoạt động đơn giản và đáng tin cậy, ổn định. Mảng các loại tiến trình và số lượng tiến trình của mỗi loại được gọi là _hệ thống tiến trình_.

ác tiến trình của ứng dụng theo 12-chuẩn [không bao giờ nên daemonize](7) hoặc viết ra các file PID. IThay vào đó, hãy dựa vào trình quản lý tiến trình của hệ điều hành (như [systemd](8), một trình quản lý tiến trình phân tán trên nền tảng đám mây hoặc một công cụ như [Foreman](9) trong phát triển) để quản lý [luồng ra][10], phản hồi các tiến trình bị lỗi, và xử lý các lần khởi động lại và tắt máy do người dùng khởi tạo.

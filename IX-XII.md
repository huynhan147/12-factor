## IX. Tính tiện dụng

### Tối đa hóa độ mạnh với khởi động nhanh và tắt máy đơn giản

  Các tiến trình của ứng dụng 12 chuẩn phải có tính khả dụng, có nghĩa là chúng có thể được bắt đầu hoặc dừng lại ở bất kỳ một thời điểm nào. Điều này tạo điều kiện cho mở rộng một cách mềm dẻo, triển khai nhanh chóng các thay đổi về code hoặc cấu hình và sự chắc chắn trong các triển khai production.

Các tiến trình nên cố gắng giảm thiểu thời gian khởi động. Lý tưởng nhất, một tiến trình mất một vài giây kể từ khi lệnh khởi chạy được thực hiện cho đến khi tiến trình này được thực hiện và sẵn sàng nhận các yêu cầu hoặc công việc. Thời gian khởi động ngắn cung cấp sự nhanh chóng và ổn định trong quá trình release và mở rộng, bởi vì người quản lý tiến trình có thể dễ dàng chuyển các tiến trình đến các máy vật lý mới khi được bảo hành.

Các tiến trình tắt máy một cách đơn giản khi chúng nhận được tín hiệu `SIGTERM` từ trình quản lý tiến trình. Đối với một tiến trình web, tắt máy đơn giản được thực hiện bằng cách ngừng lắng nghe trên cổng dịch vụ (do đó từ chối bất kỳ yêu cầu mới nào), cho phép bất kỳ yêu cầu hiện tại nào hoàn thành và sau đó thoát. Ngụ ý trong mô hình này là các yêu cầu HTTP ngắn (không quá một vài giây), hoặc trong trường hợp long-polling, client nên cố gắng kết nối lại liên tục khi kết nối bị mất.

Đối với một tiến trình worker, việc tắt máy đơn giản đạt được bằng cách trả lại công việc hiện tại cho hàng đợi công việc. Ví dụ, trên RabbitMQ, worker có thể gửi 1 `NACK`; trên Beanstalkd, công việc được trả về hàng đợi tự động bất cứ khi nào một worker ngắt kết nối. Các hệ thống dựa trên khóa như Delayed Job cần phải chắc chắn việc release khóa của chúng trong bản công việc. Ngụ ý trong mô hình này là tất cả các công việc là lặp lại, thường đạt được bằng cách gói các kết quả trong một giao dịch, hoặc làm cho hoạt động không thay đổi giá trị.
  
Các tiến trình cũng phải mạnh mẽ trước những sự cố bất ngờ , trong trường hợp có lỗi trong phần cứng cơ bản. Mặc dù đây là điều xảy hơn nhiều so với tắt máy một cách đơn giản với `SIGTERM`, nhưng nó vẫn có thể xảy ra. Cách tiếp cận được khuyến nghị là sử dụng một hệ thống queueing backend mạnh mẽ, chẳng hạn như Beanstalkd, trả về công việc cho hàng đợi khi client ngắt kết nối hoặc hết thời gian chờ. Dù bằng cách nào, ứng dụng 12-chuẩn được thiết kế để xử lý các kết thúc không mong muốn, không đẹp. Crash-only design đưa khái niệm này đến kết luận logic của nó.
## X. Dev/prod parity

### Duy trì sự tương quan/đồng giữa giai đoạn phát triển, dàn dựng và sản xuất 

Trước đây, đã có những khoảng cách đáng kể giữa giai đoạn phát triển (một nhà phát triển thực hiện chỉnh sửa trực tiếp cho triển khai cục bộ của ứng dụng) và production (một triển khai đang chạy ứng dụng được người dùng cuối truy cập). Những khoảng cách này được mô tả trong ba điểm sau:

* **Khoảng cách thời gian:** Nhà phát triển có thể làm việc trên code mất vài ngày, tuần hoặc thậm chí hàng tháng trước khi chuyển sang production.
* **Khoảng cách nhân sự**: Các nhà phát triển viết code, các kỹ sư ops triển khai nó.
* **Khoảng cách công cụ**: Các nhà phát triển có thể sử dụng một stack như Nginx, SQLite và OS X, trong khi triển khai production sử dụng Apache, MySQL và Linux.

**Ứng dụng 12-chuẩn được thiết kế để triển khai liên tục bằng cách giữ khoảng cách giữa development và production nhỏ.** Nhìn vào ba khoảng trống được mô tả ở trên:
* Làm cho khoảng cách thời gian nhỏ: một nhà phát triển có thể viết code và triển khai nó chỉ vài giờ hoặc thậm chí chỉ vài phút sau đó.
* Làm cho khoảng cách nhân sự nhỏ: các nhà phát triển viết code có liên quan chặt chẽ với việc triển khai nó và theo dõi hoạt động của nó trong production.
* Làm cho khoảng cách công cụ nhỏ: giữ cho development và production càng giống nhau càng tốt.

Tổng kết trong bảng dưới đây:


|  |  Ứng dụng truyền thống|  Ứng dụng 12 chuẩn |
| ----- |-------|-------
| Thời gian giữa các lần triển khai |  Hàng tuần |  Hàng Giờ |  
| Tác giả code và người triển khai code|  Những người khác nhau |  Người tương tự |  
| Các môi trường dev và production |  Khác nhau |  Giống nhau nhất có thể | 

Dịch vụ sao lưu, chẳng hạn như cơ sở dữ liệu của ứng dụng, hệ thống hàng đợi hoặc bộ nhớ cache, là những  thành phần rất quan trọng mà sự tương đồng giữa giai đoạn dev/prod. Nhiều ngôn ngữ cung cấp các thư viện giúp đơn giản hóa quyền truy cập vào dịch vụ sao lưu, bao gồm *adapters* cho tới các loại dịch vụ khác nhau. Một số ví dụ có trong bảng bên dưới.


| Type |  Language |  Library |  Adapters |
| ----- |-----|------|-----
| Database |  Ruby/Rails |  ActiveRecord |  MySQL, PostgreSQL, SQLite |  
| Queue |  Python/Django |  Celery |  RabbitMQ, Beanstalkd, Redis |  
| Cache |  Ruby/Rails |  ActiveSupport::Cache |  Memory, filesystem, Memcached | 

Đôi khi các nhà phát triển sẽ thích thú khi sử dụng dịch vụ sao lưu nhẹ trong môi trường local của họ, trong khi dịch vụ sao lưu ổn định và mạnh mẽ hơn sẽ được sử dụng trong prodcution. Ví dụ, sử dụng SQLite cục bộ và PostgreSQL trong production; hoặc bộ nhớ tiến trình cục bộ cho việc caching trong development và Memcached trong production.

**Nhà phát triển theo 12-chuẩn sẽ không sử dụng các dịch vụ sao lưu khác nhau giữa development và production**,ngay cả khi các adapter về mặt lý thuyết đã loại bỏ sự khác biệt trong các dịch vụ sao lưu. Sự khác biệt giữa các dịch vụ sao lưu có nghĩa là có một sự không tương thích xuất hiện, làm cho code chạy và vượt qua các kiểm tra trong quá trình development hoặc staging có thể lỗi  trong quá trình production. Các loại lỗi này tạo ra xung đột và cản trở triển khai liên tục. Ảnh hưởng của khó khăn này và những cản trở tiếp theo đến triển khai liên tục là thực sự lớn khi xem xét tổng hợp trên vòng đời của 1 ứng dụng.

Dịch vụ cục bộ nhẹ ít hấp dẫn hơn trước đây. Các dịch vụ sao lưu hiện đại như Memcached, PostgreSQL và RabbitMQ không khó cài đặt và chạy nhờ các hệ thống đóng gói hiện đại như Homebrew và apt-get. Ngoài ra, các công cụ cung cấp khai báo như Chef và Puppet kết hợp với các môi trường máy ảo nhẹ như Docker và Vagrant cho phép các nhà phát triển chạy các môi trường cục bộ tương tự với môi trường production. Chi phí cài đặt và sử dụng các hệ thống này thấp so với lợi ích của dev / prod parity và triển khai liên tục.

Các adapter cho các dịch vụ sao lưu khác nhau vẫn hữu ích, bởi vì chúng tạo cổng tới các dịch vụ sao lưu mới tương đối nhẹ nhàng. Nhưng tất cả các triển khai của ứng dụng (môi trường phát triển, dàn dựng và sản xuất ) phải sử dụng cùng loại và phiên bản của từng dịch vụ sao lưu.
## XI. Logs

### Coi các log như các luồng sự kiện

_Các log_ cho ta thấy được các hoạt động của các ứng dụng đang chạy. Trong môi trường hướng máy chủ, chúng thường được viết vào 1 file trên đĩa  (1 logfile) nhưng đây chỉ là định dạng đầu ra.

Các log là luồng sự kiện tổng hợp, được sắp xếp theo thứ tự thời gian thu thập từ những thông tin đầu ra của các tiến trình đang chạy và dịch vụ sao lưu. Log ở dạng thô thường dưới dạng văn bản với mỗi sự kiện ở mỗi dòng (mặc dù backtraces từ các ngoại lệ có thể kéo dài nhiều dòng hơn). Các log không có bắt đầu và kết thúc cố định, nhưng luồng thì liên tục khi ứng dụng đang hoạt động.

**Một ứng dụng 12-chuẩn không bao giờ quan tâm đến luồng đầu ra của routing hoặc storage.** Nó không nên thử viết vào hoặc quản lý các file log. Thay vào đó, mỗi tiến trình đang chạy sẽ tự ghi luồng sự kiện của nó, không bị chặn, để `stdout`. Trong quá trình local development, nhà phát triển sẽ thấy luồng này ở trong màn hình terminal  của họ để quan sát họat động của ứng dụng.

Trong các triển khai staging hay production, luồng của từng tiến trình sẽ được kiểm soát bởi môi trường thực thi, đối chiếu với tất cả các luồng khác từ ứng dụng và được chuyển đến một hoặc nhiều điểm đến cuối cùng để hiển thị và lưu trữ lâu dài. Những điểm đến lưu trữ này không hiển thị hoặc cấu hình bởi ứng dụng và thay vào đó được quản lý hoàn toàn bởi môi trường thực thi. Các bộ định tuyến log nguồn mở (như Logplex và Fluentd) phù hợp với mục đích này.

Luồng sự kiện cho một ứng dụng có thể được điều hướng đến một file hoặc được theo dõi thông qua lệnh tail thời gian thực trong terminal. Quan trọng nhất, luồng có thể được gửi đến một log được đánh chỉ mục và hệ thống phân tích như Splunk ,hoặc hệ thống lưu trữ dữ liệu có mục đích chung như Hadoop / Hive. Các hệ thống này mang lại sự linh hoạt và khả năng theo dõi hoạt động của ứng dụng theo thời gian, bao gồm:

* Tìm những sự kiện trong quá khứ.
* Vẽ đồ thị có quy mô lớn (chẳng hạn như yêu cầu mỗi phút).
* Kích hoạt cảnh báo dựa theo chuẩn đoán do người dùng xác định (chẳng hạn như cảnh báo khi số lượng lỗi mỗi phút vượt quá ngưỡng nhất định).
## XII. Các tiến trình admin

### Chạy các công việc admin / quản lý dưới dạng các tiến trình một lần

Hệ thống tiến trình là một mảng các tiến trình được sử dụng để thực hiện  các công việc định kỳ của ứng dụng (chẳng hạn như xử lý các yêu cầu web) khi nó chạy. Một cách riêng biệt, các nhà phát triển thường muốn thực hiện các công việc quản lý hoặc bảo trì một lần cho ứng dụng, chẳng hạn như: 

* Chạy database migrations (ví dụ: `manage.py migrate` trong Django, `rake db: migrate` trong Rails).
* Chạy một console (còn được gọi là REPL shell) để chạy code tùy ý hoặc kiểm tra các mô hình của ứng dụng dựa trên cơ sở dữ liệu thực tế. Hầu hết các ngôn ngữ cung cấp REPL bằng cách chạy trình biên dịch mà không có bất kỳ tham số nào (ví dụ: `python` hoặc `perl`) hoặc trong một số trường hợp có một lệnh riêng biệt (ví dụ: `irb` cho Ruby, `rails console` cho Rails).
* Chạy một lệnh một lần commit  lên repo của ứng dụng (ví dụ: `php scripts / fix_bad_records.php`).

Những tiến trình admin chạy một lần nên được chạy trong một môi trường giống  với môi trường chạy các tiến trình dài của ứng dụng. Chúng chạy với 1 phát hành, sẽ sử dụng chung codebase và cấu hình như bất kỳ tiến trình nào chạy với phát hành đó. Code admin phải tương thích với code ứng dụng để tránh các vấn đề đồng bộ hóa.

Các kỹ thuật tách biệt phụ thuộc nên được sử dụng trên tất cả các loại tiến trình. Ví dụ, nếu tiến trình web Ruby sử dụng lệnh `bundle exec thin start`, sau dó 1 database migration nên sử dụng `bundle exec rake db:migrate`. Tương tự như vậy, 1 chương trình Python sử dụng Virtualenv nên sử dụng vendored `bin/python` để chạy cả Tornado webserver và bất cứ tiến trình admin `manage.py` nào.

12-chuẩn đặc biệt ưa thích ngôn ngữ cung cấp REPL có hộp điều khiển, và những thứ nó dễ dàng để chạy các lệnh 1 lần. Trong triển khai cục bộ, các nh phát triển gọi các tiến trình admin chạy 1 lần bằng các lệnh shell trực tiếp trong thư mục checkout của ứng dụng. Trong 1 triển khai production, các nhà phát triển có thể sử dụng ssh hay cơ chế thực thi các lệnh remote được cung cấp bởi môi trường thực thi của triển khai để chạy 1 tiến trình.

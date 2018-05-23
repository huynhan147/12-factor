Giới thiệu
============

Trong thời đại hiện đại, phần mềm thường được phân phối dưới dạng dịch vụ: được gọi là _ứng dụng web_ hoặc phần mềm dưới dạng dịch vụ. Ứng dụng 12-chuẩn là phương pháp để xây dựng các ứng dụng phần mềm như một dịch vụ:

*   Sử dụng các định dạng **khai báo** để tự động thiết lập, giảm thiểu thời gian và chi phí cho các nhà phát triển mới tham gia dự án;
*   Có một **giao ước rõ ràng** với nền tảng hệ điều hành, cung cấp **tính di động tối đa** giữa các môi trường thực thi;
*   Phù hợp cho **triển khai** trên nền tảng đám mây **hiện đại**, giảm bớt sự cần thiết cho các máy chủ và quản trị hệ thống;
*   **Giảm thiểu sự khác biệt** giữa phát triển và sản xuất, cho phép **triển khai liên tục** m;
*   Và có thể **mở rộng quy mô** mà không thay đổi đáng kể về công cụ, kiến trúc hoặc thực tiễn phát triển.

Phương pháp 12-chuẩn có thể được áp dụng cho các ứng dụng được viết bằng bất kỳ ngôn ngữ lập trình nào và sử dụng bất kỳ kết hợp dịch vụ sao lưu nào (cơ sở dữ liệu, hàng đợi, bộ nhớ cache, v.v.).

Background
==========

Những người đóng góp cho tài liệu này đã trực tiếp tham gia vào việc phát triển và triển khai hàng trăm ứng dụng, và gián tiếp chứng kiến sự phát triển, vận hành và mở rộng hàng trăm nghìn ứng dụng thông qua công việc của chúng tôi trên nền tảng [Heroku](http://www.heroku.com/).

Tài liệu này tổng hợp tất cả các trải nghiệm và quan sát của chúng tôi về một loạt các ứng dụng phần mềm như một dịch vụ trong thực tế. Đó là một sự triangulation về phương pháp lý tưởng cho phát triển ứng dụng, đặc biệt chú ý đến sự năng động của sự phát triển hữu cơ của ứng dụng theo thời gian, động lực của sự cộng tác giữa các nhà phát triển làm việc trên codebase của ứng dụng và [tránh chi phí vận hành phần mềm]
(http://blog.heroku.com/archives/2011/6/28/the_new_heroku_4_erosion_resistance_explicit_contracts/).

Động lực của chúng tôi là nâng cao nhận thức về một số vấn đề hệ thống mà chúng tôi đã thấy trong phát triển ứng dụng hiện đại, để cung cấp vốn từ vựng chung để thảo luận những vấn đề đó và cung cấp một loạt giải pháp khái niệm rộng cho những vấn đề kèm theo thuật ngữ. Định dạng này được lấy cảm hứng từ sách của Martin Fowler _ [Các mẫu của Kiến trúc ứng dụng doanh nghiệp](https://books.google.com/books/about/Patterns_of_enterprise_application_archi.html?id=FyWZt5DdvFkC)_ và _[Tái cấu trúc](https://books.google.com/books/about/Refactoring.html?id=1MsETFPD3I0C)_.

Ai nên đọc tài liệu này?
==============================

Bất kỳ người phát triển nào đang xây dựng các ứng dụng chạy như là 1 dịch vụ. Các kĩ sư Ops triển khai hoặc điều khiển những ứng dụng như vậy.


I. Codebase
-----------

### Một codebase được theo dõi trong việc kiểm soát sửa đổi, nhiều triển khai

Ứng dụng 12-chuẩn luôn được theo dõi trong hệ thống kiểm soát phiên bản, chẳng hạn như [Git](http://git-scm.com/), [Mercurial](https://www.mercurial-scm.org/), hoặc [Subversion](http://subversion.apache.org/). Bản sao của cơ sở dữ liệu theo dõi sửa đổi được gọi là _code repository_, thường được rút ngắn thành _code repo_ hoặc chỉ _repo_.

Một _codebase_ có thể là bất kỳ repo đơn lẻ nào (trong một hệ thống kiểm soát sửa đổi tập trung như Subversion), hoặc bất kỳ bộ repo nào chia sẻ một root commit (trong một hệ thống kiểm soát sửa đổi phân cấp như Git).

Luôn có mối tương quan 1-1 giữa codebase và ứng dụng:

*   Nếu có nhiều codebase, nó không phải là một ứng dụng - đó là một hệ thống phân tán. Mỗi thành phần trong một hệ thống phân tán là một ứng dụng và mỗi thành phần có thể tuân thủ riêng với 12-chuẩn.
*  Nhiều ứng dụng chia sẻ cùng một code là vi phạm 12-chuẩn. Giải pháp ở đây là quản lý code chia sẻ thành các thư viện có thể được đưa vào thông qua [trình quản lý phụ thuộc](./dependencies).

Chỉ có một codebase cho mỗi ứng dụng, nhưng sẽ có nhiều triển khai ứng dụng.Một _triển khai_ là phiên bản đang chạy của ứng dụng. Đây thường là một trang web sản xuất và một hoặc nhiều trang web dàn dựng. Ngoài ra, mọi nhà phát triển đều có một bản sao của ứng dụng đang chạy trong môi trường phát triển local của họ, mỗi một trong số đó cũng đủ điều kiện để triển khai.

Các codebase là như nhau trên tất cả các triển khai, mặc dù các phiên bản khác nhau có thể hoạt động trong mỗi triển khai. Ví dụ, một nhà phát triển có một số commit chưa triển khai để stage; stage có một số commit chưa được triển khai tới production. Nhưng tất cả đều cùng một codebase, do đó làm cho chúng có thể nhận dạng như các triển khai khác nhau của cùng một ứng dụng.

II. Các phụ thuộc
----------------

### Khai báo rõ ràng và tách biệt các phụ thuộc

Hầu hết các ngôn ngữ lập trình đều cung cấp hệ thống đóng gói để phân phối các thư viện hỗ trợ, chẳng hạn như [CPAN](http://www.cpan.org/) cho Perl hoặc [Rubygems](http://rubygems.org/) cho Ruby. Các thư viện được cài đặt thông qua hệ thống packaging có thể được cài đặt trên toàn hệ thống (được gọi là “site package”) hoặc được đưa vào thư mục chứa ứng dụng (được gọi là “vendoring” hoặc “bundling”).

**Một ứng dụng 12-chuẩn không bao giờ dựa vào sự tồn tại ngầm của các package hệ thống.** Nó khai báo tất cả các phụ thuộc, hoàn toàn và chính xác, thông qua một khai báo _phụ thuộc_. Hơn nữa, nó sử dụng công cụ cô lập _phụ thuộc_ trong quá trình thực hiện để đảm bảo rằng không có phụ thuộc ngầm nào bị "rò rỉ" từ hệ thống xung quanh. Đặc tả phụ thuộc đầy đủ và rõ ràng được áp dụng thống nhất cho cả sản xuất và phát triển.

Ví dụ, [Bundler](https://bundler.io/) cho Ruby cung cấp định dạng biểu thị `Gemfile` cho việc khai báo các phụ thuộc và `bundle exec` cho việc cô lập phụ thuộc. Trong Python có 2 công cụ riêng biệt cho các bước này – [Pip](http://www.pip-installer.org/en/latest/) được sử dụng để khai báo và [Virtualenv](http://www.virtualenv.org/en/latest/) để cô lập. Ngay cả C cũng có [Autoconf](http://www.gnu.org/s/autoconf/) để khai báo phụ thuộc và liên kết tĩnh có thể cung cấp sự cô lập các phụ thuộc.  Chuỗi các công cụ, khai báo và cô lập phụ thuộc phải luôn luôn được sử dụng cùng nhau - chỉ có một là không đủ để đáp ứng 12-chuẩn.

Một lợi ích của khai báo phụ thuộc rõ ràng là nó đơn giản hóa việc thiết lập cho các nhà phát triển mới cho ứng dụng. Nhà phát triển mới có thể kiểm tra codebase của ứng dụng trên máy phát triển của họ, chỉ yêu cầu trình quản lý phụ thuộc và thời gian chạy ngôn ngữ được cài đặt làm điều kiện tiên quyết. Họ sẽ có thể thiết lập mọi thứ cần thiết để chạy mã của ứng dụng bằng lệnh _build command_. Ví dụ, lệnh build cho Ruby / Bundler là `bundle install`, trong khi cho Clojure / [Leiningen](https://github.com/technomancy/leiningen#readme) là `lein deps`.

Ứng dụng 12-chuẩn cũng không phụ thuộc vào sự tồn tại tiềm ẩn của bất kỳ công cụ hệ thống nào. Các ví dụ bao gồm việc kích hoạt ImageMagick hoặc `curl`. Mặc dù các công cụ này có thể tồn tại trên nhiều hoặc thậm chí hầu hết các hệ thống, không có gì đảm bảo rằng chúng sẽ tồn tại trên tất cả các hệ thống mà ứng dụng có thể chạy trong tương lai hoặc phiên bản được tìm thấy trên hệ thống tương lai sẽ tương thích với ứng dụng. Nếu ứng dụng cần một công cụ hệ thống, công cụ đó sẽ được đưa vào ứng dụng.

III. Cấu hình
-----------

### Lưu cấu hình trong môi trường

 _Cấu hình_ một ứng dụng là mọi thứ có khả năng thay đổi giữa [triển khai](./codebase) (môi trường staging, môi trường thực thi, môi trường nhà phát triển, etc). Nó bao gồm:

*   Tài nguyên xử lý cơ sở dữ liệu, Memcached và [dịch vụ sao lưu khác](./backing-services)
*   Thông tin đăng nhập cho các dịch vụ bên ngoài như Amazon S3 hoặc Twitter
*   Các giá trị của mỗi triển khai như là tên máy chủ của triển khai

Đôi khi, các ứng dụng lưu trữ cấu hình dưới dạng hằng số trong code. Đây là vi phạm 12-chuẩn, yêu cầu **tách cấu hình nghiêm ngặt khỏi code**. Cấu hình khác nhau đáng kể trên các triển khai, code thì không.

Một bài kiểm tra litmus xem liệu một ứng dụng có tất cả các cấu hình một cách chính xác với code là xác định liệu codebase có thể được làm nguồn mở tại bất kỳ thời điểm nào, mà không ảnh hưởng đến bất kỳ thông tin xác thực nào.

Lưu ý rằng định nghĩa "cấu hình" này không **không** bao gồm cấu hình ứng dụng nội bộ, chẳng hạn như `config / routes.rb` trong Rails, hoặc cách [module  code được kết nối](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html) trong [Spring](http://spring.io/). Loại cấu hình này không thay đổi giữa các triển khai và do đó được thực hiện tốt nhất trong code.

Một cách tiếp cận khác để cấu hình là sử dụng các file cấu hình không được kiểm tra trong điều khiển sửa đổi, chẳng hạn như `config / database.yml` trong Rails. Đây là một cải tiến lớn so với việc sử dụng các hằng số được kiểm tra trong mã repo, nhưng vẫn có điểm yếu: rất dễ nhầm lẫn khi kiểm tra file cấu hình cho repo; có một xu hướng cho các tập tin cấu hình được phân tán ở những nơi khác nhau và các định dạng khác nhau, làm cho nó khó khăn để xem và quản lý tất cả các cấu hình ở một nơi. Hơn nữa, các định dạng này có xu hướng đặc biệt về ngôn ngữ hoặc framework.

**Ứng dụng 12-chuẩn lưu trữ cấu hình trong các _biến môi trường_** (thường đọc tắt là _env vars_ hoặc _env_). Env vars có thể dễ dàng thay đổi giữa các triển khai mà không phải thay đổi code; không giống như các file cấu hình, khả năng chúng được thêm vào code repo một cách vô tình là rất thấp; và không giống như các file cấu hình thông thường, các cơ chế cấu hình khác như Java System Properties, chúng là các chuẩn ngôn ngữ và OS-agnostic.

Một khía cạnh khác của quản lý cấu hình là nhóm. Đôi khi, ứng dụng định cấu hình hàng loạt thành các nhóm được đặt tên (thường được gọi là "môi trường") được đặt tên theo các triển khai cụ thể, chẳng hạn như môi trường `development`,` test` và `production` trong Rails. Phương pháp này không quy mô rõ ràng: vì nhiều triển khai ứng dụng được tạo, các tên môi trường mới là cần thiết, chẳng hạn như `staging` hoặc` qa`. Khi dự án phát triển hơn nữa, các nhà phát triển có thể thêm các môi trường đặc biệt của riêng mình như `joes-staging`, kết quả của việc kết hợp các cấu hình này làm cho việc quản lý triển khai ứng dụng trở nên mong manh.

Trong một ứng dụng 12-chuẩn, env vars là điều khiển chi tiết, mỗi chúng đều độc lập với các env vars khác. Chúng không bao giờ được nhóm lại với nhau thành “môi trường”, mà thay vào đó được quản lý độc lập cho từng triển khai. Đây là một mô hình có quy mô thuận lợi khi ứng dụng mở rộng nhiều triển khai hơn suốt lifetime của nó.

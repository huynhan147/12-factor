
## IV. Dịch vụ sao lưu

### Coi dịch vụ sao lưu như tài nguyên đính kèm

Một _dịch vụ sao lưu_ là bất kỳ dịch vụ nào mà ứng dụng sử dụng kết nối mạng như là một phần trong quá trình họat động bình thường của nó . Các ví dụ bao gồm kho dữ liệu (như [MySQL](1) hoặc [CouchDB](2)), các hệ thống nhắn tin/xếp hàng (như [RabbitMQ](3) or [Beanstalkd](4)), các dịch vụ STMP cho gửi mail (such as [Postfix](5)), và các hệ thống caching (như [Memcached][6]).

Các dịch vụ sao lưu như cơ sở dữ liệu thường được quản lý bới các quản trị viên hệ thống như là các triển khai trong quá trình chạy ứng dụng. Ngoài các dịch vụ được quản lý cục bộ này, ứng dụng cũng có thể có các dịch vụ được cung cấp và quản lý bới bên thứ 3. Ví dụ bao gôm dịch vụ STMP (như [Postmark](7)), dịch vụ thu thập số liệu (như [New Relic](8) hoặc (Loggly)[9]), dịch vụ binary asset (như  [Amazon S3](10)), và thậm chí cả dịch vụ sử dụng API-accessible (như [Twitter](11), [Google Maps](12), or [Last.fm](13)).

**Code ứng dụng theo 12-chuẩn không phân biệt dịch vụ cục bộ hay dịch vụ bên thứ ba.** Đối với ứng dụng,cả hai đều là tài nguyên đính kèm, được truy cập qua URL hoặc các định vị/thông tin xác thực khác được lưu trong [cấu hình](14). Một bản [triển khai](15) của ứng dụng 12-chuẩn có thể hoán đổi một cơ sở dữ liệu MySQL cục bộ với một cơ sở dữ liệu được quản lý bởi bên thứ ba (như [Amazon RDS][16]) mà không có bất kỳ thay đổi nào đối với code của ứng dụng. Tương tự như vậy, một máy chủ STMP cục bộ có thể hoán đổi với dịch vụ STMP của bên thứ ba (như Postmark) mà không phải thay đổi code. Trong cả hai trường hợp, chỉ tài nguyên xử lý trong cấu hình cần được thay đổi.

Mỗi dịch vụ sao lưu riêng biệt là một _tài nguyên_. Ví dụ, một cơ sở dữ liệu MySQL là một tài nguyên; hai cơ sở dữ liệu MySQL (được sử dụng để sharding ở tầng ứng dụng) đủ điều kiện là hai tài nguyên riêng biệt. Ứng dụng 12-chuẩn coi các cơ sở dữ liệu này như  _các tài nguyên đính kèm_, cho biết kết nối lỏng lẻo của chúng với triển khai mà chúng được gắn vào.


Tài nguyên có thể được gắn vào và tách ra khỏi các triển khai theo ý muốn. Ví dụ: nếu cơ sở dữ liệu của ứng dụng bị lỗi do sự cố phần cứng, người quản trị ứng dụng có thể chuyển qua một máy chủ cơ sở dữ liệu mới được khôi phục từ bản sao lưu gần đây. Cơ sở dữ liệu production hiện tại có thể được tách ra và cơ sở dữ liệu mới được thêm vào - tất cả đều không có bất kỳ thay đổi code nào.
## V. Xây dựng, xuất bản, chạy

### Tách biệt giai đoạn xây dựng và chạy

Một [codebase](1) được chuyển thành một bản triển khai (không phát triển) qua ba giai đoạn:

* _Giai đoạn xây dựng_ là một quá trình chuyển đổi một code repo thành một gói thực thi được gọi là một _build_ . Using a version of the code at a commit specified by the deployment process, the build stage fetches vendors [dependencies][2] and compiles binaries and assets.
* The _release stage_ takes the build produced by the build stage and combines it with the deploy's current [config][3]. The resulting _release_ contains both the build and the config and is ready for immediate execution in the execution environment.
* The _run stage_ (also known as "runtime") runs the app in the execution environment, by launching some set of the app's [processes][4] against a selected release.

![Code becomes a build, which is combined with config to create a release.][5]

**The twelve-factor app uses strict separation between the build, release, and run stages.** For example, it is impossible to make changes to the code at runtime, since there is no way to propagate those changes back to the build stage.

Deployment tools typically offer release management tools, most notably the ability to roll back to a previous release. For example, the [Capistrano][6] deployment tool stores releases in a subdirectory named `releases`, where the current release is a symlink to the current release directory. Its `rollback` command makes it easy to quickly roll back to a previous release.

Every release should always have a unique release ID, such as a timestamp of the release (such as `2011-04-06-20:32:17`) or an incrementing number (such as `v100`). Releases are an append-only ledger and a release cannot be mutated once it is created. Any change must create a new release.

Builds are initiated by the app's developers whenever new code is deployed. Runtime execution, by contrast, can happen automatically in cases such as a server reboot, or a crashed process being restarted by the process manager. Therefore, the run stage should be kept to as few moving parts as possible, since problems that prevent an app from running can cause it to break in the middle of the night when no developers are on hand. The build stage can be more complex, since errors are always in the foreground for a developer who is driving the deploy.
## VI. Processes

### Execute the app as one or more stateless processes

The app is executed in the execution environment as one or more _processes_.

In the simplest case, the code is a stand-alone script, the execution environment is a developer's local laptop with an installed language runtime, and the process is launched via the command line (for example, `python my_script.py`). On the other end of the spectrum, a production deploy of a sophisticated app may use many [process types, instantiated into zero or more running processes][1].

**Twelve-factor processes are stateless and [share-nothing][2].** Any data that needs to persist must be stored in a stateful [backing service][3], typically a database.

The memory space or filesystem of the process can be used as a brief, single-transaction cache. For example, downloading a large file, operating on it, and storing the results of the operation in the database. The twelve-factor app never assumes that anything cached in memory or on disk will be available on a future request or job – with many processes of each type running, chances are high that a future request will be served by a different process. Even when running only one process, a restart (triggered by code deploy, config change, or the execution environment relocating the process to a different physical location) will usually wipe out all local (e.g., memory and filesystem) state.

Asset packagers like [django-assetpackager][4] use the filesystem as a cache for compiled assets. A twelve-factor app prefers to do this compiling during the [build stage][5]. Asset packagers such as [Jammit][6] and the [Rails asset pipeline][7] can be configured to package assets during the build stage.

Some web systems rely on ["sticky sessions"][8] – that is, caching user session data in memory of the app's process and expecting future requests from the same visitor to be routed to the same process. Sticky sessions are a violation of twelve-factor and should never be used or relied upon. Session state data is a good candidate for a datastore that offers time-expiration, such as [Memcached][9] or [Redis][10].
## VII. Port binding

### Export services via port binding

Web apps are sometimes executed inside a webserver container. For example, PHP apps might run as a module inside [Apache HTTPD][1], or Java apps might run inside [Tomcat][2].

**The twelve-factor app is completely self-contained** and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service. The web app **exports HTTP as a service by binding to a port**, and listening to requests coming in on that port.

In a local development environment, the developer visits a service URL like `http://localhost:5000/` to access the service exported by their app. In deployment, a routing layer handles routing requests from a public-facing hostname to the port-bound web processes.

This is typically implemented by using [dependency declaration][3] to add a webserver library to the app, such as [Tornado][4] for Python, [Thin][5] for Ruby, or [Jetty][6] for Java and other JVM-based languages. This happens entirely in _user space_, that is, within the app's code. The contract with the execution environment is binding to a port to serve requests.

HTTP is not the only service that can be exported by port binding. Nearly any kind of server software can be run via a process binding to a port and awaiting incoming requests. Examples include [ejabberd][7] (speaking [XMPP][8]), and [Redis][9] (speaking the [Redis protocol][10]).

Note also that the port-binding approach means that one app can become the [backing service][11] for another app, by providing the URL to the backing app as a resource handle in the [config][12] for the consuming app.
## VIII. Concurrency

### Scale out via the process model

Any computer program, once run, is represented by one or more processes. Web apps have taken a variety of process-execution forms. For example, PHP processes run as child processes of Apache, started on demand as needed by request volume. Java processes take the opposite approach, with the JVM providing one massive uberprocess that reserves a large block of system resources (CPU and memory) on startup, with concurrency managed internally via threads. In both cases, the running process(es) are only minimally visible to the developers of the app.

![Scale is expressed as running processes, workload diversity is expressed as process types.][1]

**In the twelve-factor app, processes are a first class citizen.** Processes in the twelve-factor app take strong cues from [the unix process model for running service daemons][2]. Using this model, the developer can architect their app to handle diverse workloads by assigning each type of work to a _process type_. For example, HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process.

This does not exclude individual processes from handling their own internal multiplexing, via threads inside the runtime VM, or the async/evented model found in tools such as [EventMachine][3], [Twisted][4], or [Node.js][5]. But an individual VM can only grow so large (vertical scale), so the application must also be able to span multiple processes running on multiple physical machines.

The process model truly shines when it comes time to scale out. The [share-nothing, horizontally partitionable nature of twelve-factor app processes][6] means that adding more concurrency is a simple and reliable operation. The array of process types and number of processes of each type is known as the _process formation_.

Twelve-factor app processes [should never daemonize][7] or write PID files. Instead, rely on the operating system's process manager (such as [systemd][8], a distributed process manager on a cloud platform, or a tool like [Foreman][9] in development) to manage [output streams][10], respond to crashed processes, and handle user-initiated restarts and shutdowns.

## IX. Disposability

### Maximize robustness with fast startup and graceful shutdown

**The twelve-factor app's [processes](1) are _disposable_, meaning they can be started or stopped at a moment's notice.** This facilitates fast elastic scaling, rapid deployment of [code](2) or [config](3) changes, and robustness of production deploys.

Processes should strive to **minimize startup time**. Ideally, a process takes a few seconds from the time the launch command is executed until the process is up and ready to receive requests or jobs. Short startup time provides more agility for the [release][4] process and scaling up; and it aids robustness, because the process manager can more easily move processes to new physical machines when warranted.

Processes **shut down gracefully when they receive a [SIGTERM][5]** signal from the process manager. For a web process, graceful shutdown is achieved by ceasing to listen on the service port (thereby refusing any new requests), allowing any current requests to finish, and then exiting. Implicit in this model is that HTTP requests are short (no more than a few seconds), or in the case of long polling, the client should seamlessly attempt to reconnect when the connection is lost.

For a worker process, graceful shutdown is achieved by returning the current job to the work queue. For example, on [RabbitMQ][6] the worker can send a [`NACK`][7]; on [Beanstalkd][8], the job is returned to the queue automatically whenever a worker disconnects. Lock-based systems such as [Delayed Job][9] need to be sure to release their lock on the job record. Implicit in this model is that all jobs are [reentrant][10], which typically is achieved by wrapping the results in a transaction, or making the operation [idempotent][11].

Processes should also be **robust against sudden death**, in the case of a failure in the underlying hardware. While this is a much less common occurrence than a graceful shutdown with `SIGTERM`, it can still happen. A recommended approach is use of a robust queueing backend, such as Beanstalkd, that returns jobs to the queue when clients disconnect or time out. Either way, a twelve-factor app is architected to handle unexpected, non-graceful terminations. [Crash-only design][12] takes this concept to its [logical conclusion][13].
## X. Dev/prod parity

### Keep development, staging, and production as similar as possible

Historically, there have been substantial gaps between development (a developer making live edits to a local [deploy][1] of the app) and production (a running deploy of the app accessed by end users). These gaps manifest in three areas:

* **The time gap:** A developer may work on code that takes days, weeks, or even months to go into production.
* **The personnel gap**: Developers write code, ops engineers deploy it.
* **The tools gap**: Developers may be using a stack like Nginx, SQLite, and OS X, while the production deploy uses Apache, MySQL, and Linux.

**The twelve-factor app is designed for [continuous deployment][2] by keeping the gap between development and production small.** Looking at the three gaps described above:
* Make the time gap small: a developer may write code and have it deployed hours or even just minutes later.
* Make the personnel gap small: developers who wrote code are closely involved in deploying it and watching its behavior in production.
* Make the tools gap small: keep development and production as similar as possible.

Summarizing the above into a table:


|  |  Traditional app |  Twelve-factor app |
| ----- |-------|-------
| Time between deploys |  Weeks |  Hours |  
| Code authors vs code deployers |  Different people |  Same people |  
| Dev vs production environments |  Divergent |  As similar as possible | 

[Backing services][3], such as the app's database, queueing system, or cache, is one area where dev/prod parity is important. Many languages offer libraries which simplify access to the backing service, including _adapters_ to different types of services. Some examples are in the table below.


| Type |  Language |  Library |  Adapters |
| ----- |-----|------|-----
| Database |  Ruby/Rails |  ActiveRecord |  MySQL, PostgreSQL, SQLite |  
| Queue |  Python/Django |  Celery |  RabbitMQ, Beanstalkd, Redis |  
| Cache |  Ruby/Rails |  ActiveSupport::Cache |  Memory, filesystem, Memcached | 

Developers sometimes find great appeal in using a lightweight backing service in their local environments, while a more serious and robust backing service will be used in production. For example, using SQLite locally and PostgreSQL in production; or local process memory for caching in development and Memcached in production.

**The twelve-factor developer resists the urge to use different backing services between development and production**, even when adapters theoretically abstract away any differences in backing services. Differences between backing services mean that tiny incompatibilities crop up, causing code that worked and passed tests in development or staging to fail in production. These types of errors create friction that disincentivizes continuous deployment. The cost of this friction and the subsequent dampening of continuous deployment is extremely high when considered in aggregate over the lifetime of an application.

Lightweight local services are less compelling than they once were. Modern backing services such as Memcached, PostgreSQL, and RabbitMQ are not difficult to install and run thanks to modern packaging systems, such as [Homebrew][4] and [apt-get][5]. Alternatively, declarative provisioning tools such as [Chef][6] and [Puppet][7] combined with light-weight virtual environments such as [Docker][8] and [Vagrant][9] allow developers to run local environments which closely approximate production environments. The cost of installing and using these systems is low compared to the benefit of dev/prod parity and continuous deployment.

Adapters to different backing services are still useful, because they make porting to new backing services relatively painless. But all deploys of the app (developer environments, staging, production) should be using the same type and version of each of the backing services.
## XI. Logs

### Treat logs as event streams

_Logs_ provide visibility into the behavior of a running app. In server-based environments they are commonly written to a file on disk (a "logfile"); but this is only an output format.

Logs are the [stream][1] of aggregated, time-ordered events collected from the output streams of all running processes and backing services. Logs in their raw form are typically a text format with one event per line (though backtraces from exceptions may span multiple lines). Logs have no fixed beginning or end, but flow continuously as long as the app is operating.

**A twelve-factor app never concerns itself with routing or storage of its output stream.** It should not attempt to write to or manage logfiles. Instead, each running process writes its event stream, unbuffered, to `stdout`. During local development, the developer will view this stream in the foreground of their terminal to observe the app's behavior.

In staging or production deploys, each process' stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. These archival destinations are not visible to or configurable by the app, and instead are completely managed by the execution environment. Open-source log routers (such as [Logplex][2] and [Fluentd][3]) are available for this purpose.

The event stream for an app can be routed to a file, or watched via realtime tail in a terminal. Most significantly, the stream can be sent to a log indexing and analysis system such as [Splunk][4], or a general-purpose data warehousing system such as [Hadoop/Hive][5]. These systems allow for great power and flexibility for introspecting an app's behavior over time, including:

* Finding specific events in the past.
* Large-scale graphing of trends (such as requests per minute).
* Active alerting according to user-defined heuristics (such as an alert when the quantity of errors per minute exceeds a certain threshold).
## XII. Admin processes

### Run admin/management tasks as one-off processes

The [process formation][1] is the array of processes that are used to do the app's regular business (such as handling web requests) as it runs. Separately, developers will often wish to do one-off administrative or maintenance tasks for the app, such as:

* Running database migrations (e.g. `manage.py migrate` in Django, `rake db:migrate` in Rails).
* Running a console (also known as a [REPL][2] shell) to run arbitrary code or inspect the app's models against the live database. Most languages provide a REPL by running the interpreter without any arguments (e.g. `python` or `perl`) or in some cases have a separate command (e.g. `irb` for Ruby, `rails console` for Rails).
* Running one-time scripts committed into the app's repo (e.g. `php scripts/fix_bad_records.php`).

One-off admin processes should be run in an identical environment as the regular [long-running processes][3] of the app. They run against a [release][4], using the same [codebase][5] and [config][6] as any process run against that release. Admin code must ship with application code to avoid synchronization issues.

The same [dependency isolation][7] techniques should be used on all process types. For example, if the Ruby web process uses the command `bundle exec thin start`, then a database migration should use `bundle exec rake db:migrate`. Likewise, a Python program using Virtualenv should use the vendored `bin/python` for running both the Tornado webserver and any `manage.py` admin processes.

Twelve-factor strongly favors languages which provide a REPL shell out of the box, and which make it easy to run one-off scripts. In a local deploy, developers invoke one-off admin processes by a direct shell command inside the app's checkout directory. In a production deploy, developers can use ssh or other remote command execution mechanism provided by that deploy's execution environment to run such a process.

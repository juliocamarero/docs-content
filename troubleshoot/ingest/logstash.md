---
navigation_title: "Logstash"
mapped_pages:
  - https://www.elastic.co/guide/en/logstash/current/troubleshooting.html
  - https://www.elastic.co/guide/en/logstash/current/ts-logstash.html
applies_to:
  stack: ga
  serverless: ga
---

# Troubleshoot Logstash [ts-logstash]

% What needs to be done: Lift-and-shift

% Use migrated content from existing pages that map to this page:

% - [ ] ./raw-migrated-files/logstash/logstash/troubleshooting.md
% - [ ] ./raw-migrated-files/logstash/logstash/ts-logstash.md

This page helps you troubleshoot Logstash.


## Installation and setup [ts-install]

### Inaccessible temp directory [ts-temp-dir]

Certain versions of the JRuby runtime and libraries in certain plugins (the Netty network library in the TCP input, for example) copy executable files to the temp directory. This situation causes subsequent failures when `/tmp` is mounted `noexec`.

**Sample error**

```sh
[2018-03-25T12:23:01,149][ERROR][org.logstash.Logstash ]
java.lang.IllegalStateException: org.jruby.exceptions.RaiseException:
(LoadError) Could not load FFI Provider: (NotImplementedError) FFI not
available: java.lang.UnsatisfiedLinkError: /tmp/jffi5534463206038012403.so:
/tmp/jffi5534463206038012403.so: failed to map segment from shared object:
Operation not permitted
```

**Possible solutions**

* Change setting to mount `/tmp` with `exec`.
* Specify an alternate directory using the `-Djava.io.tmpdir` setting in the `jvm.options` file.



## {{ls}} start up [ts-startup]

### *Illegal reflective access* errors [ts-illegal-reflective-error]

After an upgrade, Logstash may show warnings similar to these:

```sh
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.jruby.ext.openssl.SecurityHelper (file:/{...}/jruby{...}jopenssl.jar) to field java.security.MessageDigest.provider
WARNING: Please consider reporting this to the maintainers of org.jruby.ext.openssl.SecurityHelper
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```

These errors appear related to [a known issue with JRuby](https://github.com/jruby/jruby/issues/4834).

**Work around**

Try adding these values to the `jvm.options` file.

```sh
--add-opens=java.base/java.security=ALL-UNNAMED
--add-opens=java.base/java.io=ALL-UNNAMED
--add-opens=java.base/java.nio.channels=ALL-UNNAMED
--add-opens=java.base/sun.nio.ch=org.ALL-UNNAMED
--add-opens=java.management/sun.management=ALL-UNNAMED
```

**Notes:**

* These settings allow Logstash to start without warnings.
* This workaround has been tested with simple pipelines. If you have experiences to share, comment in the [issue](https://github.com/elastic/logstash/issues/10496).


### *Permission denied - NUL* errors on Windows [ts-windows-permission-denied-NUL]

Logstash may not start with some user-supplied versions of the JDK on Windows.

**Sample error**

```sh
[FATAL] 2022-04-27 15:13:16.650 [main] Logstash - Logstash stopped processing because of an error: (EACCES) Permission denied - NUL
org.jruby.exceptions.SystemCallError: (EACCES) Permission denied - NUL
```

This error appears to be related to a [JDK issue](https://bugs.openjdk.java.net/browse/JDK-8285445) where a new property was added with an inappropriate default.

This issue affects some OpenJDK-derived JVM versions (Adoptium, OpenJDK, and Azul Zulu) on Windows:

* `11.0.15+10`
* `17.0.3+7`

**Work around**

* Use the [bundled JDK](logstash://reference/getting-started-with-logstash.md#ls-jvm) included with Logstash
* Or, try adding this value to the `jvm.options` file, and restarting Logstash

    ```sh
    -Djdk.io.File.enableADS=true
    ```



### Container exits with *An unexpected error occurred!* message [ts-container-cgroup]

{{ls}} running in a container may not start due to a [bug in the JDK](https://bugs.openjdk.org/browse/JDK-8343191).

**Sample error**

```sh
[FATAL] 2024-11-11 11:11:11.465 [LogStash::Runner] runner - An unexpected error occurred! {:error=>#<Java::JavaLang::NullPointerException: >, :backtrace=>[
        "java.util.Objects.requireNonNull(java/util/Objects.java:233)",
        "sun.nio.fs.UnixFileSystem.getPath(sun/nio/fs/UnixFileSystem.java:296)",
        "java.nio.file.Path.of(java/nio/file/Path.java:148)",
        "java.nio.file.Paths.get(java/nio/file/Paths.java:69)",
        "jdk.internal.platform.CgroupUtil.lambda$readStringValue$1(jdk/internal/platform/CgroupUtil.java:67)",
        "java.security.AccessController.doPrivileged(java/security/AccessController.java:571)",
        "jdk.internal.platform.CgroupUtil.readStringValue(jdk/internal/platform/CgroupUtil.java:69)",
        "jdk.internal.platform.CgroupSubsystemController.getStringValue(jdk/internal/platform/CgroupSubsystemController.java:65)",
        "jdk.internal.platform.cgroupv1.CgroupV1Subsystem.getCpuSetCpus(jdk/internal/platform/cgroupv1/CgroupV1Subsystem.java:275)",
        "jdk.internal.platform.CgroupMetrics.getCpuSetCpus(jdk/internal/platform/CgroupMetrics.java:100)",
        "com.sun.management.internal.OperatingSystemImpl.isCpuSetSameAsHostCpuSet(com/sun/management/internal/OperatingSystemImpl.java:277)",
        "com.sun.management.internal.OperatingSystemImpl$ContainerCpuTicks.getContainerCpuLoad(com/sun/management/internal/OperatingSystemImpl.java:96)",
        "com.sun.management.internal.OperatingSystemImpl.getProcessCpuLoad(com/sun/management/internal/OperatingSystemImpl.java:271)",
        "org.logstash.instrument.monitors.ProcessMonitor$Report.<init>(org/logstash/instrument/monitors/ProcessMonitor.java:63)",
        "org.logstash.instrument.monitors.ProcessMonitor.detect(org/logstash/instrument/monitors/ProcessMonitor.java:136)",
        "org.logstash.instrument.reports.ProcessReport.generate(org/logstash/instrument/reports/ProcessReport.java:35)",
        "jdk.internal.reflect.DirectMethodHandleAccessor.invoke(jdk/internal/reflect/DirectMethodHandleAccessor.java:103)",
        "java.lang.reflect.Method.invoke(java/lang/reflect/Method.java:580)",
        "org.jruby.javasupport.JavaMethod.invokeDirectWithExceptionHandling(org/jruby/javasupport/JavaMethod.java:300)",
        "org.jruby.javasupport.JavaMethod.invokeStaticDirect(org/jruby/javasupport/JavaMethod.java:222)",
        "RUBY.collect_process_metrics(/usr/share/logstash/logstash-core/lib/logstash/instrument/periodic_poller/jvm.rb:102)",
        "RUBY.collect(/usr/share/logstash/logstash-core/lib/logstash/instrument/periodic_poller/jvm.rb:73)",
        "RUBY.start(/usr/share/logstash/logstash-core/lib/logstash/instrument/periodic_poller/base.rb:72)",
        "org.jruby.RubySymbol$SymbolProcBody.yieldSpecific(org/jruby/RubySymbol.java:1541)",
        "org.jruby.RubySymbol$SymbolProcBody.doYield(org/jruby/RubySymbol.java:1534)",
        "org.jruby.RubyArray.collectArray(org/jruby/RubyArray.java:2770)",
        "org.jruby.RubyArray.map(org/jruby/RubyArray.java:2803)",
        "org.jruby.RubyArray$INVOKER$i$0$0$map.call(org/jruby/RubyArray$INVOKER$i$0$0$map.gen)",
        "RUBY.start(/usr/share/logstash/logstash-core/lib/logstash/instrument/periodic_pollers.rb:41)",
        "RUBY.configure_metrics_collectors(/usr/share/logstash/logstash-core/lib/logstash/agent.rb:477)",
        "RUBY.initialize(/usr/share/logstash/logstash-core/lib/logstash/agent.rb:88)",
        "org.jruby.RubyClass.new(org/jruby/RubyClass.java:949)",
        "org.jruby.RubyClass$INVOKER$i$newInstance.call(org/jruby/RubyClass$INVOKER$i$newInstance.gen)",
        "RUBY.create_agent(/usr/share/logstash/logstash-core/lib/logstash/runner.rb:552)",
        "RUBY.execute(/usr/share/logstash/logstash-core/lib/logstash/runner.rb:434)",
        "RUBY.run(/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/clamp-1.0.1/lib/clamp/command.rb:68)",
        "RUBY.run(/usr/share/logstash/logstash-core/lib/logstash/runner.rb:293)",
        "RUBY.run(/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/clamp-1.0.1/lib/clamp/command.rb:133)",
        "usr.share.logstash.lib.bootstrap.environment.<main>(/usr/share/logstash/lib/bootstrap/environment.rb:89)",
        "usr.share.logstash.lib.bootstrap.environment.run(usr/share/logstash/lib/bootstrap//usr/share/logstash/lib/bootstrap/environment.rb)",
        "java.lang.invoke.MethodHandle.invokeWithArguments(java/lang/invoke/MethodHandle.java:733)",
        "org.jruby.Ruby.runScript(org/jruby/Ruby.java:1245)",
        "org.jruby.Ruby.runNormally(org/jruby/Ruby.java:1157)",
        "org.jruby.Ruby.runFromMain(org/jruby/Ruby.java:983)",
        "org.logstash.Logstash.run(org/logstash/Logstash.java:163)",
        "org.logstash.Logstash.main(org/logstash/Logstash.java:73)"
    ]
}
[FATAL] 2024-11-11 11:11:11.516 [LogStash::Runner] Logstash - Logstash stopped processing because of an error: (SystemExit) exit
    org.jruby.exceptions.SystemExit: (SystemExit) exit
    at org.jruby.RubyKernel.exit(org/jruby/RubyKernel.java: 921) ~[jruby.jar:?]
    at org.jruby.RubyKernel.exit(org/jruby/RubyKernel.java: 880) ~[jruby.jar:?]
    at usr.share.logstash.lib.bootstrap.environment.<main>(/usr/share/logstash/lib/bootstrap/environment.rb: 90) ~[?:?]
```

This error can happen when cgroups v2 is not enabled, such as when running on a Red Had version 8 operating system.

**Work around**

Follow your operating system’s instructions for enabling cgroups v2.



## Troubleshooting persistent queues [ts-pqs]

Symptoms of persistent queue problems include {{ls}} or one or more pipelines not starting successfully, accompanied by an error message similar to this one.

```
message=>"java.io.IOException: Page file size is too small to hold elements"
```

See the [troubleshooting information](logstash://reference/persistent-queues.md#troubleshooting-pqs) in the persistent queue section for more information on remediating problems with persistent queues.


## Data ingestion [ts-ingest]

### Error response code 429 [ts-429]

A `429` message indicates that an application is busy handling other requests. For example, Elasticsearch sends a `429` code to notify Logstash (or other indexers) that the bulk failed because the ingest queue is full. Logstash will retry sending documents.

**Possible actions**

Check {{es}} to see if it needs attention.

* [Cluster stats API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-stats)
* [Monitoring](/deploy-manage/monitor.md)

**Sample error**

```
[2018-08-21T20:05:36,111][INFO ][logstash.outputs.elasticsearch] retrying
failed action with response code: 429
({"type"=>"es_rejected_execution_exception", "reason"=>"rejected execution of
org.elasticsearch.transport.TransportService$7@85be457 on
EsThreadPoolExecutor[bulk, queue capacity = 200,
org.elasticsearch.common.util.concurrent.EsThreadPoolExecutor@538c9d8a[Running,
pool size = 16, active threads = 16, queued tasks = 200, completed tasks =
685]]"})
```



## Performance [ts-performance]

For general performance tuning tips and guidelines, see [*Performance tuning*](logstash://reference/performance-tuning.md).


## Troubleshooting a pipeline [ts-pipeline]

Pipelines, by definition, are unique. Here are some guidelines to help you get started.

* Identify the offending pipeline.
* Start small. Create a minimum pipeline that manifests the problem.

For basic pipelines, this configuration could be enough to make the problem show itself.

```ruby
input {stdin{}} output {stdout{}}
```

{{ls}} can separate logs by pipeline. This feature can help you identify the offending pipeline. Set `pipeline.separate_logs: true` in your `logstash.yml` to enable the log per pipeline feature.

For more complex pipelines, the problem could be caused by a series of plugins in a specific order. Troubleshooting these pipelines usually requires trial and error. Start by systematically removing input and output plugins until you’re left with the minimum set that manifest the issue.

We want to expand this section to make it more helpful. If you have troubleshooting tips to share:

* create an issue at [https://github.com/elastic/logstash/issues](https://github.com/elastic/logstash/issues), or
* create a pull request with your proposed changes at [https://github.com/elastic/logstash](https://github.com/elastic/logstash).


## Logging level can affect performances [ts-pipeline-logging-level-performance]

**Symptoms**

Simple filters such as `mutate` or `json` filter can take several milliseconds per event to execute. Inputs and outputs might be affected, too.

**Background**

The different plugins running on Logstash can be quite verbose if the logging level is set to `debug` or `trace`. As the logging library used in Logstash is synchronous, heavy logging can affect performances.

**Solution**

Reset the logging level to `info`.


## Logging in json format can write duplicate `message` fields [ts-pipeline-logging-json-duplicated-message-field]

**Symptoms**

When log format is `json` and certain log events (for example errors from JSON codec plugin) contains two instances of the `message` field.

Without setting this flag, json log would contain objects like:

```json
{
   "level":"WARN",
   "loggerName":"logstash.codecs.jsonlines",
   "timeMillis":1712937761955,
   "thread":"[main]<stdin",
   "logEvent":{
      "message":"JSON parse error, original data now in message field",
      "message":"Unexpected close marker '}': expected ']' (for Array starting at [Source: (String)\"{\"name\": [}\"; line: 1, column: 10])\n at [Source: (String)\"{\"name\": [}\"; line: 1, column: 12]",
      "exception":"LogStash::Json::ParserError",
      "data":"{\"name\": [}"
   }
}
```

Note the duplication of `message` field, while being technically valid json, it is not always parsed correctly.

**Solution** In `config/logstash.yml` enable the strict json flag:

```yaml
log.format.json.fix_duplicate_message_fields: true
```

or pass the command line switch

```
bin/logstash --log.format.json.fix_duplicate_message_fields true
```

With `log.format.json.fix_duplicate_message_fields` enabled the duplication of `message` field is removed, adding to the field name a `_1` suffix:

```json
{
   "level":"WARN",
   "loggerName":"logstash.codecs.jsonlines",
   "timeMillis":1712937629789,
   "thread":"[main]<stdin",
   "logEvent":{
      "message":"JSON parse error, original data now in message field",
      "message_1":"Unexpected close marker '}': expected ']' (for Array starting at [Source: (String)\"{\"name\": [}\"; line: 1, column: 10])\n at [Source: (String)\"{\"name\": [}\"; line: 1, column: 12]",
      "exception":"LogStash::Json::ParserError",
      "data":"{\"name\": [}"
   }
}
```
## Additional resources

* [Troubleshoot Logstash plugins](/troubleshoot/ingest/logstash/plugins.md)
* [Logstash: Contribute and discuss](/troubleshoot/ingest/logstash/contribute.md)
* [Troubleshoot: Contact us](/troubleshoot/index.md#contact-us)




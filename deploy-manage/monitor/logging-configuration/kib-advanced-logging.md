---
mapped_pages:
  - https://www.elastic.co/guide/en/kibana/current/_cli_configuration.html
applies_to:
  deployment:
    self:
    ece:
    eck:
---

# Advanced {{kib}} logging settings

You do not need to configure any additional settings to use the logging features in {{kib}}. Logging is enabled by default, and will log at info level using the `pattern` layout, which outputs logs to `stdout`.

If you are planning to ingest your logs using {{es}} or another tool, we recommend using the `json` layout, which produces logs in ECS format. In general, `pattern` layout is recommended when raw logs will be read by a human, and `json` layout when logs will be read by a machine.

:::{note}
You can't configure these settings in an {{ech}} deployment.
:::

The {{kib}} logging system has three main components: *loggers*, *appenders* and *layouts*.

* **Loggers** define what logging settings should be applied to a particular logger.
* [Appenders](#logging-appenders) define where log messages are displayed (for example, stdout or console) and stored (for example, file on the disk).
* [Layouts](#logging-layouts) define how log messages are formatted and what type of information they include.


These components allow us to log messages according to message type and level, to control how these messages are formatted and where the final logs will be displayed or stored.

* [Log level](#log-level)
* [Layouts](#logging-layouts)
* [Logger hierarchy](#logger-hierarchy)

:::{tip}
For additional information about the available logging settings, refer to the [{{kib}} configuration reference](kibana://reference/configuration-reference/logging-settings.md).
:::

## Log level [log-level]

{{kib}} logging supports the following log levels: `off`, `fatal`, `error`, `warn`, `info`, `debug`, `trace`, `all`.

Levels are ordered, so `off` > `fatal` > `error` > `warn` > `info` > `debug` > `trace` > `all`.

A log record will be logged by the logger if its level is higher than or equal to the level of its logger. For example: If the output of an API call is configured to log at the `info` level and the parameters passed to the API call are set to `debug`, with a global logging configuration in `kibana.yml` set to `debug`, both the output *and* parameters are logged. If the log level is set to `info`, the debug logs are ignored, meaning that you’ll only get a record for the API output and *not* for the parameters.

Logging set at a plugin level is always respected, regardless of the `root` logger level. In other words, if root logger is set to fatal and pluginA logging is set to `debug`, debug logs are only shown for pluginA, with other logs only reporting on `fatal`.

The `all` and `off` levels can only be used in configuration, and are handy shortcuts that allow you to log every log record or disable logging entirely for a specific logger. These levels can also be specified using [CLI arguments](#logging-cli-migration).


## Layouts [logging-layouts]

Every appender should know exactly how to format log messages before they are written to the console or file on the disk. This behavior is controlled by the layouts and configured through `appender.layout` configuration property for every custom appender. Currently we don’t define any default layout for the custom appenders, so one should always make the choice explicitly.

There are two types of layout supported at the moment: [`pattern`](#pattern-layout) and [`json`](#json-layout).

### Pattern layout [pattern-layout]

With `pattern` layout, it’s possible to define a string pattern with special placeholders `%conversion_pattern` that will be replaced with data from the actual log message. By default, the following pattern is used: `[%date][%level][%logger] %message`.

::::{note}
The `pattern` layout uses a sub-set of [log4j 2 pattern syntax](https://logging.apache.org/log4j/2.x/manual/layouts.html#PatternLayout) and doesn’t implement all `log4j 2` capabilities.
::::

The following conversions are provided out of the box:

* **level**: Outputs the [level](/deploy-manage/monitor/logging-configuration/kibana-log-levels.md) of the logging event. Example of `%level` output: `TRACE`, `DEBUG`, `INFO`.

* **logger**: Outputs the name of the logger that published the logging event. Example of `%logger` output: `server`, `server.http`, `server.http.kibana`.

* **message**: Outputs the application supplied message associated with the logging event.

* **meta**: Outputs the entries of `meta` object data in **json** format, if one is present in the event. Example of `%meta` output:

    ```bash
    // Meta{from: 'v7', to: 'v8'}
    '{"from":"v7","to":"v8"}'
    // Meta empty object
    '{}'
    // no Meta provided
    ''
    ```

$$$date-format$$$
* **date**: Outputs the date of the logging event. The date conversion specifier may be followed by a set of braces containing a name of predefined date format and canonical timezone name. Timezone name is expected to be one from [TZ database name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). Timezone defaults to the host timezone when not explicitly specified. Example of `%date` output:

    $$$date-conversion-pattern-examples$$$

    | Conversion pattern | Example |
    | --- | --- |
    | `%date` | `2012-02-01T14:30:22.011Z` uses `ISO8601` format by default |
    | `%date{{ISO8601}}` | `2012-02-01T14:30:22.011Z` |
    | `%date{{ISO8601_TZ}}` | `2012-02-01T09:30:22.011-05:00`   `ISO8601` with timezone |
    | `%date{{ISO8601_TZ}}{America/Los_Angeles}` | `2012-02-01T06:30:22.011-08:00` |
    | `%date{{ABSOLUTE}}` | `09:30:22.011` |
    | `%date{{ABSOLUTE}}{America/Los_Angeles}` | `06:30:22.011` |
    | `%date{{UNIX}}` | `1328106622` |
    | `%date{{UNIX_MILLIS}}` | `1328106622011` |

* **pid**: Outputs the process ID.

The pattern layout also offers a `highlight` option that allows you to highlight some parts of the log message with different colors. Highlighting is quite handy if log messages are forwarded to a terminal with color support.


### JSON layout [json-layout]

With `json` layout log messages will be formatted as JSON strings in [ECS format](ecs://reference/index.md) that includes a timestamp, log level, logger, message text and any other metadata that may be associated with the log message itself.


## Logger hierarchy [logger-hierarchy]

Every logger has a unique name that follows a hierarchical naming rule. The logger is considered to be an ancestor of another logger if its name followed by a `.` is a prefix of the descendant logger. For example, a logger named `a.b` is an ancestor of logger `a.b.c`. All top-level loggers are descendants of a special `root` logger at the top of the logger hierarchy. The `root` logger always exists, is fully configured and logs to `info` level by default. The `root` logger must also be configured if any other logging configuration is specified in your `kibana.yml`.

You can configure *[log level](/deploy-manage/monitor/logging-configuration/kibana-log-levels.md)* and *appenders* for a specific logger. If a logger only has a *log level* configured, then the *appenders* configuration applied to the logger is inherited from the ancestor logger, up to the `root` logger.

::::{note}
In the current implementation we *don’t support* so called *appender additivity* when log messages are forwarded to *every* distinct appender within the ancestor chain including `root`. That means that log messages are only forwarded to appenders that are configured for a particular logger. If a logger doesn’t have any appenders configured, the configuration of that particular logger will be inherited from its closest ancestor.
::::

### Dedicated loggers [dedicated-loggers]

#### Root

The `root` logger has a dedicated configuration node since this logger is special and should always exist. By default `root` is configured with `info` level and `default` appender that is also always available. This is the configuration that all custom loggers will use unless they’re re-configured explicitly.

For example to see *all* log messages that fall back on the `root` logger configuration, just add one line to the configuration:

```yaml
logging.root.level: all
```

Or disable logging entirely with `off`:

```yaml
logging.root.level: off
```

#### Metrics logs

The `metrics.ops` logger is configured with `debug` level and will automatically output sample system and process information at a regular interval. The metrics that are logged are a subset of the data collected and are formatted in the log message as follows:

| Ops formatted log property | Location in metrics service | Log units |
| --- | --- | --- |
| memory | process.memory.heap.used_in_bytes | [depends on the value](http://numeraljs.com/#format), typically MB or GB |
| uptime | process.uptime_in_millis | HH:mm:ss |
| load | os.load | [ "load for the last 1 min" "load for the last 5 min" "load for the last 15 min"] |
| delay | process.event_loop_delay | ms |

The log interval is the same as the interval at which system and process information is refreshed and is configurable under `ops.interval`:

```yaml
ops.interval: 5000
```

The minimum interval is 100ms and defaults to 5000ms.


#### Request and response logs [request-response-logger]

The `http.server.response` logger is configured with `debug` level and will automatically output data about http requests and responses occurring on the {{kib}} server. The message contains some high-level information, and the corresponding log meta contains the following:

| Meta property | Description | Format |
| --- | --- | --- |
| client.ip | IP address of the requesting client | ip |
| http.request.method | http verb for the request (uppercase) | string |
| http.request.mime_type | (optional) mime as specified in the headers | string |
| http.request.referrer | (optional) referrer | string |
| http.request.headers | request headers | object |
| http.response.body.bytes | (optional) Calculated response payload size in bytes | number |
| http.response.status_code | status code returned | number |
| http.response.headers | response headers | object |
| http.response.responseTime | (optional) Calculated response time in ms | number |
| url.path | request path | string |
| url.query | (optional) request query string | string |
| user_agent.original | raw user-agent string provided in request headers | string |


## Appenders [logging-appenders]


### Rolling file appender [rolling-file-appender]

Similar to Log4j’s `RollingFileAppender`, this appender will log into a file, and rotate it following a rolling strategy when the configured policy triggers.


#### Triggering policies [_triggering_policies]

The triggering policy determines when a rollover should occur.

There are currently two policies supported: `size-limit` and `time-interval`.

##### Size-limit triggering policy [size-limit-triggering-policy]

This policy will rotate the file when it reaches a predetermined size.

```yaml
logging:
  appenders:
    rolling-file:
      type: rolling-file
      fileName: /var/logs/kibana.log
      policy:
        type: size-limit
        size: 50mb
      strategy:
        //...
      layout:
        type: pattern
```

The options are:

* `size`: The maximum size the log file should reach before a rollover should be performed. The default value is `100mb`.


##### Time-interval triggering policy [time-interval-triggering-policy]

This policy will rotate the file every given interval of time.

```yaml
logging:
  appenders:
    rolling-file:
      type: rolling-file
      fileName: /var/logs/kibana.log
      policy:
        type: time-interval
        interval: 10s
        modulate: true
      strategy:
        //...
      layout:
        type: pattern
```

The options are:

* `interval`: How often a rollover should occur. The default value is `24h`.

* `modulate`: Whether the interval should be adjusted to cause the next rollover to occur on the interval boundary.

    For example, if modulate is true and the interval is `4h`, if the current hour is 3 am then the first rollover will occur at 4 am and then next ones will occur at 8 am, noon, 4pm, etc. The default value is `true`.


#### Rolling strategies [_rolling_strategies]

The rolling strategy determines how the rollover should occur: both the naming of the rolled files, and their retention policy.

There is currently one strategy supported: `numeric`.

**Numeric rolling strategy**

This strategy will suffix the file with a given pattern when rolling, and will retains a fixed amount of rolled files.

```yaml
logging:
  appenders:
    rolling-file:
      type: rolling-file
      fileName: /var/logs/kibana.log
      policy:
        // ...
      strategy:
        type: numeric
        pattern: '-%i'
        max: 2
      layout:
        type: pattern
```

For example, with this configuration:

* During the first rollover `kibana.log` is renamed to `kibana-1.log`. A new `kibana.log` file is created and starts being written to.
* During the second rollover `kibana-1.log` is renamed to `kibana-2.log` and `kibana.log` is renamed to `kibana-1.log`. A new `kibana.log` file is created and starts being written to.
* During the third and subsequent rollovers, `kibana-2.log` is deleted, kibana-1.log is renamed to `kibana-2.log` and `kibana.log` is renamed to kibana-1.log. A new `kibana.log` file is created and starts being written to.

The options are:

* `pattern`: The suffix to append to the file path when rolling. Must include `%i`, as this is the value that will be converted to the file index.

    For example, with `fileName: /var/logs/kibana.log` and `pattern: '-%i'`, the rolling files created will be `/var/logs/kibana-1.log`, `/var/logs/kibana-2.log`, and so on. The default value is `-%i`

* `max`: The maximum number of files to keep. Once this number is reached, oldest files will be deleted. The default value is `7`


### Rewrite appender [rewrite-appender]

::::{warning}
This appender is currently considered experimental and is not intended for public consumption. The API is subject to change at any time.
::::


Similar to log4j’s `RewriteAppender`, this appender serves as a sort of middleware, modifying the provided log events before passing them along to another appender.

```yaml
logging:
  appenders:
    my-rewrite-appender:
      type: rewrite
      appenders: [console, file] # name of "destination" appender(s)
      policy:
        # ...
```

The most common use case for the `RewriteAppender` is when you want to filter or censor sensitive data that may be contained in a log entry. In fact, with a default configuration, {{kib}} will automatically redact any `authorization`, `cookie`, or `set-cookie` headers when logging http requests & responses.

To configure additional rewrite rules, you’ll need to specify a [`RewritePolicy`](#rewrite-policies).


#### Rewrite policies [rewrite-policies]

Rewrite policies exist to indicate which parts of a log record can be modified within the rewrite appender.

##### Meta

The `meta` rewrite policy can read and modify any data contained in the `LogMeta` before passing it along to a destination appender.

Meta policies must specify one of three modes, which indicate which action to perform on the configured properties: - `update` updates an existing property at the provided `path`. - `remove` removes an existing property at the provided `path`.

The `properties` are listed as a `path` and `value` pair, where `path` is the dot-delimited path to the target property in the `LogMeta` object, and `value` is the value to add or update in that target property. When using the `remove` mode, a `value` is not necessary.

Here’s an example of how you would replace any `cookie` header values with `[REDACTED]`:

```yaml
logging:
  appenders:
    my-rewrite-appender:
      type: rewrite
      appenders: [console]
      policy:
        type: meta # indicates that we want to rewrite the LogMeta
        mode: update # will update an existing property only
        properties:
          - path: "http.request.headers.cookie" # path to property
            value: "[REDACTED]" # value to replace at path
```

Rewrite appenders can even be passed to other rewrite appenders to apply multiple filter policies/modes, as long as it doesn’t create a circular reference. Each rewrite appender is applied sequentially (one after the other).

```yaml
logging:
  appenders:
    remove-request-headers:
      type: rewrite
      appenders: [censor-response-headers] # redirect to the next rewrite appender
      policy:
        type: meta
        mode: remove
        properties:
          - path: "http.request.headers" # remove all request headers
    censor-response-headers:
      type: rewrite
      appenders: [console] # output to console
      policy:
        type: meta
        mode: update
        properties:
          - path: "http.response.headers.set-cookie"
            value: "[REDACTED]"
```


##### Rewrite appender configuration example [_complete_example_for_rewrite_appender]

```yaml
logging:
  appenders:
    custom_console:
      type: console
      layout:
        type: pattern
        highlight: true
        pattern: "[%date][%level][%logger] %message %meta"
    file:
      type: file
      fileName: ./kibana.log
      layout:
        type: json
    censor:
      type: rewrite
      appenders: [custom_console, file]
      policy:
        type: meta
        mode: update
        properties:
          - path: "http.request.headers.cookie"
            value: "[REDACTED]"
  loggers:
    - name: http.server.response
      appenders: [censor] # pass these logs to our rewrite appender
      level: debug
```

## Logging configuration using the CLI [logging-cli-migration]

You can specify your logging configuration using the CLI. For convenience, the `--verbose` and `--silent` flags exist as shortcuts and will continue to be supported beyond v7.

If you wish to override these flags, you can always do so by passing your preferred logging configuration directly to the CLI. For example, with the following configuration:

```yaml
logging:
  appenders:
    custom:
      type: console
      layout:
        type: pattern
        pattern: "[%date][%level] %message"
  root:
    level: warn
    appenders: [custom]
```

you can override the root logging level using the following flags:

| Legacy logging | {{kib}} platform logging | CLI shortcuts |
| --- | --- | --- |
| --verbose | --logging.root.level=debug | --verbose |
| --silent | --logging.root.level=off | --silent |

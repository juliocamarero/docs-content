---
mapped_pages:
  - https://www.elastic.co/guide/en/observability/current/apm-data-model-errors.html
applies_to:
  stack:
  serverless:
---

# Errors [apm-data-model-errors]

An error event contains at least information about the original `exception` that occurred or about a `log` created when the exception occurred. For simplicity, errors are represented by a unique ID.

An Error contains:

* Both the captured `exception` and the captured `log` of an error can contain a `stack trace`, which is helpful for debugging.
* The `culprit` of an error indicates where it originated.
* An error might relate to the [transaction](/solutions/observability/apm/transactions.md) during which it happened, via the `transaction.id`.
* Data about the environment in which the event is recorded:

    * Service - environment, framework, language, etc.
    * Host - architecture, hostname, IP, etc.
    * Process - args, PID, PPID, etc.
    * URL - full, domain, port, query, etc.
    * [User](/solutions/observability/apm/metadata.md#apm-data-model-user) - (if supplied) email, ID, username, etc.

In addition, agents provide options for users to capture custom [metadata](/solutions/observability/apm/metadata.md). Metadata can be indexed - [`labels`](/solutions/observability/apm/metadata.md#apm-data-model-labels), or not-indexed - [`custom`](/solutions/observability/apm/metadata.md#apm-data-model-custom).

::::{tip}
Most agents limit keyword fields (e.g. `error.id`) to 1024 characters, non-keyword fields (e.g. `error.exception.message`) to 10,000 characters.
::::

Errors are stored in error indices.

## Data streams [_data_streams_3]

Errors are stored in the following data streams:

* APM error/exception logging: `logs-apm.error-<namespace>`
* Applications UI logging: `logs-apm.app.<service.name>-<namespace>`

See [Data streams](/solutions/observability/apm/data-streams.md) to learn more.

## Example error document [_example_error_document]

This example shows what error documents can look like when indexed in {{es}}.

::::{dropdown} Expand {{es}} document
```json
[
    {
        "@timestamp": "2017-05-09T15:04:05.999Z",
        "agent": {
            "name": "elastic-node",
            "version": "3.14.0"
        },
        "container": {
            "id": "container-id"
        },
        "ecs": {
            "version": "1.12.0"
        },
        "error": {
            "grouping_key": "d6b3f958dfea98dc9ed2b57d5f0c48bb",
            "grouping_name": "Cannot read property 'baz' of undefined",
            "id": "0f0e9d67c1854d21a6f44673ed561ec8",
            "log": {
                "level": "custom log level",
                "message": "Cannot read property 'baz' of undefined"
            }
        },
        "event": {
            "ingested": "2020-04-22T14:52:08.436124Z"
        },
        "host": {
            "architecture": "x64",
            "ip": ["127.0.0.1"],
            "os": {
                "platform": "darwin"
            }
        },
        "kubernetes": {
            "namespace": "namespace1",
            "pod": {
                "name": "pod-name",
                "uid": "pod-uid"
            }
        },
        "labels": {
            "tag1": "one",
            "tag2": 2
        },
        "message": "Cannot read property 'baz' of undefined",
        "observer": {
            "hostname": "ix.lan",
            "type": "apm-server",
            "version": "8.0.0"
        },
        "process": {
            "args": [
                "node",
                "server.js"
            ],
            "parent": {
                "pid": 7788
            },
            "pid": 1234,
            "title": "node"
        },
        "processor": {
            "event": "error",
            "name": "error"
        },
        "service": {
            "environment": "staging",
            "framework": {
                "name": "Express",
                "version": "1.2.3"
            },
            "language": {
                "name": "ecmascript",
                "version": "8"
            },
            "name": "1234_service-12a3",
            "node": {
                "name": "myservice-node"
            },
            "runtime": {
                "name": "node",
                "version": "8.0.0"
            },
            "version": "5.1.3"
        },
        "timestamp": {
            "us": 1494342245999000
        }
    },
    {
        "@timestamp": "2017-05-09T15:04:05.999Z",
        "agent": {
            "name": "python",
            "version": "4.3"
        },
        "client": {
            "geo": {
                "continent_name": "North America",
                "country_iso_code": "US",
                "country_name": "United States",
                "location": {
                    "lat": 37.751,
                    "lon": -97.822
                }
            },
            "ip": "8.8.8.8"
        },
        "container": {
            "id": "container-id"
        },
        "ecs": {
            "version": "1.12.0"
        },
        "error": {
            "culprit": "my.module.function_name",
            "custom": {
                "and_objects": {
                    "foo": [
                        "bar",
                        "baz"
                    ]
                },
                "my_key": 1,
                "some_other_value": "foo bar"
            },
            "exception": [
                {
                    "attributes": {
                        "foo": "bar"
                    },
                    "code": "42",
                    "handled": false,
                    "message": "The username root is unknown",
                    "module": "__builtins__",
                    "stacktrace": [
                        {
                            "abs_path": "/real/file/name.py",
                            "context": {
                                "post": [
                                    "line4",
                                    "line5"
                                ],
                                "pre": [
                                    "line1",
                                    "line2"
                                ]
                            },
                            "exclude_from_grouping": false,
                            "filename": "file/name.py",
                            "function": "foo",
                            "library_frame": true,
                            "line": {
                                "column": 4,
                                "context": "line3",
                                "number": 3
                            },
                            "module": "App::MyModule",
                            "vars": {
                                "key": "value"
                            }
                        },
                        {
                            "abs_path": "/Users/watson/code/node_modules/elastic/lib/instrumentation/index.js",
                            "context": {
                                "post": [
                                    "    ins.currentTransaction = prev",
                                    "    return result",
                                    "}",
                                    "}",
                                    "",
                                    "Instrumentation.prototype._recoverTransaction = function (trans) {",
                                    "  if (this.currentTransaction === trans) return"
                                ],
                                "pre": [
                                    "  var trans = this.currentTransaction",
                                    "",
                                    "  return instrumented",
                                    "",
                                    "  function instrumented () {",
                                    "    var prev = ins.currentTransaction",
                                    "    ins.currentTransaction = trans"
                                ]
                            },
                            "exclude_from_grouping": false,
                            "filename": "lib/instrumentation/index.js",
                            "function": "instrumented",
                            "line": {
                                "context": "    var result = original.apply(this, arguments)",
                                "number": 102
                            },
                            "vars": {
                                "key": "value"
                            }
                        }
                    ],
                    "type": "DbError"
                }
            ],
            "grouping_key": "50f62f37edffc4630c6655ba3ecfcf46",
            "grouping_name": "My service could not talk to the database named foobar",
            "id": "5f0e9d64c1854d21a6f44673ed561ec8",
            "log": {
                "level": "warning",
                "logger_name": "my.logger.name",
                "message": "My service could not talk to the database named foobar",
                "param_message": "My service could not talk to the database named %s",
                "stacktrace": [
                    {
                        "abs_path": "/real/file/name.py",
                        "context": {
                            "post": [
                                "line4",
                                "line5"
                            ],
                            "pre": [
                                "line1",
                                "line2"
                            ]
                        },
                        "exclude_from_grouping": false,
                        "filename": "/webpack/file/name.py",
                        "function": "foo",
                        "line": {
                            "column": 4,
                            "context": "line3",
                            "number": 3
                        },
                        "module": "App::MyModule",
                        "vars": {
                            "key": "value"
                        }
                    },
                    {
                        "abs_path": "/Users/watson/code/node_modules/elastic/lib/instrumentation/index.js",
                        "context": {
                            "post": [
                                "    ins.currentTransaction = prev",
                                "    return result",
                                "}",
                                "}",
                                "",
                                "Instrumentation.prototype._recoverTransaction = function (trans) {",
                                "  if (this.currentTransaction === trans) return"
                            ],
                            "pre": [
                                "  var trans = this.currentTransaction",
                                "",
                                "  return instrumented",
                                "",
                                "  function instrumented () {",
                                "    var prev = ins.currentTransaction",
                                "    ins.currentTransaction = trans"
                            ]
                        },
                        "exclude_from_grouping": false,
                        "filename": "lib/instrumentation/index.js",
                        "function": "instrumented",
                        "line": {
                            "context": "    var result = original.apply(this, arguments)",
                            "number": 102
                        },
                        "vars": {
                            "key": "value"
                        }
                    }
                ]
            }
        },
        "event": {
            "ingested": "2020-04-22T14:52:08.384032Z"
        },
        "host": {
            "architecture": "x64",
            "ip": "127.0.0.1",
            "os": {
                "platform": "darwin"
            }
        },
        "http": {
            "request": {
                "body": {
                    "original": "Hello World"
                },
                "cookies": {
                    "c1": "v1",
                    "c2": "v2"
                },
                "env": {
                    "GATEWAY_INTERFACE": "CGI/1.1",
                    "SERVER_SOFTWARE": "nginx"
                },
                "headers": {
                    "Array": [
                        "foo",
                        "bar",
                        "baz"
                    ],
                    "Content-Type": [
                        "text/html"
                    ],
                    "Cookie": [
                        "c1=v1,c2=v2"
                    ],
                    "Some-Other-Header": [
                        "foo"
                    ],
                    "User-Agent": [
                        "Mozilla Chrome Edge"
                    ]
                },
                "method": "POST",
                "referrer": "http://localhost:8000/test/e2e/"
            },
            "response": {
                "finished": true,
                "headers": {
                    "Content-Type": [
                        "application/json"
                    ]
                },
                "headers_sent": true,
                "status_code": 200
            },
            "version": "1.1"
        },
        "kubernetes": {
            "namespace": "namespace1",
            "pod": {
                "name": "pod-name",
                "uid": "pod-uid"
            }
        },
        "labels": {
            "organization_uuid": "9f0e9d64-c185-4d21-a6f4-4673ed561ec8",
            "tag1": "one",
            "tag2": 2
        },
        "message": "My service could not talk to the database named foobar",
        "observer": {
            "ephemeral_id": "f1838cde-80dd-4af5-b7ac-ffc2d3fccc9d",
            "hostname": "ix.lan",
            "id": "5d4dc8fe-cb14-47ee-b720-d6bf49f87ef0",
            "type": "apm-server",
            "version": "8.0.0"
        },
        "process": {
            "args": [
                "node",
                "server.js"
            ],
            "pid": 1234,
            "parent": {
                "pid": 7788
            },
            "title": "node"
        },
        "processor": {
            "event": "error",
            "name": "error"
        },
        "service": {
            "environment": "staging",
            "framework": {
                "name": "Express",
                "version": "1.2.3"
            },
            "language": {
                "name": "ecmascript",
                "version": "8"
            },
            "name": "abc",
            "node": {
                "name": "myservice-xz"
            },
            "runtime": {
                "name": "node",
                "version": "1.2"
            },
            "version": "5.1.3"
        },
        "source": {
            "ip": "8.8.8.8"
        },
        "timestamp": {
            "us": 1494342245999999
        },
        "url": {
            "domain": "www.example.com",
            "fragment": "#hash",
            "full": "https://www.example.com/p/a/t/h?query=string#hash",
            "original": "/p/a/t/h?query=string#hash",
            "path": "/p/a/t/h",
            "port": 8080,
            "query": "?query=string",
            "scheme": "https"
        },
        "user": {
            "email": "foo@example.com",
            "id": "99",
            "name": "foo"
        },
        "user_agent": {
            "device": {
                "name": "Other"
            },
            "name": "Other",
            "original": "Mozilla Chrome Edge"
        }
    },
    {
        "@timestamp": "2017-05-09T15:04:05.000Z",
        "agent": {
            "name": "elastic-node",
            "version": "3.14.0"
        },
        "container": {
            "id": "container-id"
        },
        "ecs": {
            "version": "1.12.0"
        },
        "error": {
            "exception": [
                {
                    "type": "connection error"
                }
            ],
            "grouping_key": "18f82051862e494727fa20e0adc15711",
            "grouping_name": null,
            "id": "7f0e9d68c1854d21a6f44673ed561ec8"
        },
        "event": {
            "ingested": "2020-04-22T14:52:08.435669Z"
        },
        "host": {
            "architecture": "x64",
            "ip": "127.0.0.1",
            "os": {
                "platform": "darwin"
            }
        },
        "kubernetes": {
            "namespace": "namespace1",
            "pod": {
                "name": "pod-name",
                "uid": "pod-uid"
            }
        },
        "labels": {
            "tag1": "one",
            "tag2": 2
        },
        "observer": {
            "ephemeral_id": "f1838cde-80dd-4af5-b7ac-ffc2d3fccc9d",
            "hostname": "ix.lan",
            "id": "5d4dc8fe-cb14-47ee-b720-d6bf49f87ef0",
            "type": "apm-server",
            "version": "8.0.0"
        },
        "process": {
            "args": [
                "node",
                "server.js"
            ],
            "pid": 1234,
            "parent": {
                "pid": 7788
            },
            "title": "node"
        },
        "processor": {
            "event": "error",
            "name": "error"
        },
        "service": {
            "environment": "staging",
            "framework": {
                "name": "Express",
                "version": "1.2.3"
            },
            "language": {
                "name": "ecmascript",
                "version": "8"
            },
            "name": "1234_service-12a3",
            "node": {
                "name": "myservice-node"
            },
            "runtime": {
                "name": "node",
                "version": "8.0.0"
            },
            "version": "5.1.3"
        },
        "timestamp": {
            "us": 1494342245000000
        }
    },
    {
        "@timestamp": "2017-05-09T15:04:05.000Z",
        "agent": {
            "name": "elastic-node",
            "version": "3.14.0"
        },
        "container": {
            "id": "container-id"
        },
        "ecs": {
            "version": "1.12.0"
        },
        "error": {
            "exception": [
                {
                    "code": "35",
                    "message": "foo is not defined"
                }
            ],
            "grouping_key": "f6b5a2877d9b00d5b32b44c9db039f11",
            "grouping_name": "foo is not defined",
            "id": "8f0e9d68c1854d21a6f44673ed561ec8"
        },
        "event": {
            "ingested": "2020-04-22T14:52:08.435117Z"
        },
        "host": {
            "architecture": "x64",
            "ip": "127.0.0.1",
            "os": {
                "platform": "darwin"
            }
        },
        "kubernetes": {
            "namespace": "namespace1",
            "pod": {
                "name": "pod-name",
                "uid": "pod-uid"
            }
        },
        "labels": {
            "tag1": "one",
            "tag2": 2
        },
        "message": "foo is not defined",
        "observer": {
            "ephemeral_id": "f1838cde-80dd-4af5-b7ac-ffc2d3fccc9d",
            "hostname": "ix.lan",
            "id": "5d4dc8fe-cb14-47ee-b720-d6bf49f87ef0",
            "type": "apm-server",
            "version": "8.0.0"
        },
        "process": {
            "args": [
                "node",
                "server.js"
            ],
            "pid": 1234,
            "parent": {
                "pid": 7788
            },
            "title": "node"
        },
        "processor": {
            "event": "error",
            "name": "error"
        },
        "service": {
            "environment": "staging",
            "framework": {
                "name": "Express",
                "version": "1.2.3"
            },
            "language": {
                "name": "ecmascript",
                "version": "8"
            },
            "name": "1234_service-12a3",
            "node": {
                "name": "myservice-node"
            },
            "runtime": {
                "name": "node",
                "version": "8.0.0"
            },
            "version": "5.1.3"
        },
        "timestamp": {
            "us": 1494342245000000
        }
    }
]
```

::::


---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-search-queries.html
applies_to:
  deployment:
    ess: all
    ece: all
    eck: all
    self: all
  serverless: unavailable
---

# Audit {{es}} search queries [auditing-search-queries]

There is no [audit event type](elasticsearch://reference/elasticsearch/elasticsearch-audit-events.md) specifically dedicated to search queries. Search queries are analyzed and then processed; the processing triggers authorization actions that are audited. However, the original raw query, as submitted by the client, is not accessible downstream when authorization auditing occurs.

Search queries are contained inside HTTP request bodies, however, and some audit events that are generated by the REST layer, on the coordinating node, can be toggled to output the request body to the audit log. Therefore, one must audit request bodies in order to audit search queries.

To make certain audit events include the request body, configure the following setting in {{es}}:

```yaml
xpack.security.audit.logfile.events.emit_request_body: true
```

You can apply this setting through [cluster update settings API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-put-settings), as described in [](./configuring-audit-logs.md). Alternatively, you can modify `elasticsearch.yml` in all nodes and restart for the changes to take effect.

::::{important}
No filtering is performed when auditing, so sensitive data might be audited in plain text when audit events include the request body. Also, the request body can contain malicious content that can break a parser consuming the audit logs.
::::

The request body is printed as an escaped JSON string value (RFC 4627) to the `request.body` event attribute.

Not all events contain the `request.body` attribute, even when the above setting is toggled. The ones that do are:

* `authentication_success`
* `authentication_failed`
* `realm_authentication_failed`
* `tampered_request`
* `run_as_denied`
* `anonymous_access_denied`

The `request.body` attribute is printed on the coordinating node only (the node that handles the REST request). Most of these event types are [not included by default](elasticsearch://reference/elasticsearch/configuration-reference/auding-settings.md#xpack-sa-lf-events-include).

A good practical piece of advice is to add `authentication_success` to the event types that are audited (add it to the list in the `xpack.security.audit.logfile.events.include`), as this event type is not audited by default.

::::{note}
Typically, the include list contains other event types as well, such as `access_granted` or `access_denied`.
::::



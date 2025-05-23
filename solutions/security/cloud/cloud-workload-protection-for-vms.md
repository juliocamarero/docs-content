---
mapped_pages:
  - https://www.elastic.co/guide/en/security/current/cloud-workload-protection.html
  - https://www.elastic.co/guide/en/serverless/current/security-cloud-workload-protection.html
applies_to:
  stack: all
  serverless:
    security: all
---

# Cloud workload protection for VMs


Cloud workload protection helps you monitor and protect your Linux VMs. It uses the [{{elastic-defend}}](/solutions/security/configure-elastic-defend/install-elastic-defend.md) integration to capture cloud workload telemetry containing process, file, and network activity.

Use this telemetry with out-of-the-box detection rules and machine learning models to automate processes that identify cloud threats.


## Use cases [_use_cases]

* **Runtime monitoring of cloud workloads:** Provides visibility into cloud workloads, context for detected threats, and the historical data needed for retroactive threat investigations.
* **Cloud-native threat detection and prevention:** Provides security coverage for Linux, containers, and serverless applications. Protects against known and unknown threats using on-host detections and protections against malicious behavior, memory threats, and malware.
* **Reducing the time to detect and remediate runtime threats:** Helps you resolve potential threats by showing alerts in context, making the data necessary for further investigations readily available, and providing remediation options.

To continue setting up your cloud workload protection, learn more about:

* [**Getting started with {{elastic-defend}}**](/solutions/security/configure-elastic-defend/install-elastic-defend.md): configure {{elastic-defend}} to protect your hosts. Be sure to select one of the "Cloud workloads" presets if you want to collect session data by default, including process, file, and network telemetry.
* [**Session view**](/solutions/security/investigate/session-view.md): examine Linux process data organized in a tree-like structure according to the Linux logical event model, with processes organized by parentage and time of execution. Use it to monitor and investigate session activity, and to understand user and service behavior on your Linux infrastructure.
* [**Environment variable capture**](/solutions/security/cloud/capture-environment-variables.md): Capture the environment variables associated with process events, such as `PATH`, `LD_PRELOAD`, or `USER`.

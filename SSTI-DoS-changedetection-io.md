# SSTI leading to Denial of Service in changedetection.io

## Overview

* **Vulnerability Type**: Server-Side Template Injection (SSTI)
* **Vendor**: dgtlmoon
* **Product**: [changedetection.io](https://github.com/dgtlmoon/changedetection.io)
* **Affected Version**: <= 0.51.4
* **Component**: Notification Title field

## Description

A Server-Side Template Injection (SSTI) vulnerability exists in `changedetection.io` (versions <= 0.51.4), allowing a remote attacker to cause a Denial of Service (DoS) via resource exhaustion.

The vulnerability stems from unsafe processing of user-supplied data within the Jinja2 template engine. While the application implements a `SandboxedEnvironment` to prevent Remote Code Execution (RCE) by blocking unsafe attributes, it fails to restrict computational complexity or memory usage for standard operations.

An attacker can inject malicious Python expressions (specifically exponential string multiplication) into the **Notification Title** field. When the application renders this template, the Python process attempts to allocate an abnormal amount of memory, leading to an application crash or a complete host system instability (OOM).

## Steps to Reproduce

1. **Setup**: Deploy a local instance of `changedetection.io` (e.g., using Docker).
2. **Navigate**: Go to the **Settings** -> **Notifications** tab.
3. **Inject**: In the **Notification Title** input field, insert the following Jinja2 payload:
```python
{{ "a" * 10000000000 }}

```


4. **Trigger**: Click the **"Save"** or **"Send Test Notification"** button.
5. **Observe**:
   * The web request will hang indefinitely (browser status: "Waiting for localhost...").
   * As shown in the attached screenshots comparison:
     * **Normal State**: The application operates with minimal resource usage.
     * **Attack State**: Immediately after submitting the form with the payload, the docker stats output reveals a critical spike in resource consumption. The container's MEM USAGE rapidly escalated to over 37 GB, while CPU usage saturated the core, confirming that the application is exhausting host resources.
<img width="2024" height="794" alt="图片" src="https://github.com/user-attachments/assets/8ed46756-b951-425d-9a79-81c99bab1366" />
<img width="2005" height="831" alt="图片" src="https://github.com/user-attachments/assets/1764d3bf-0d86-48a5-bf5c-4672fe642f6c" />



## Impact

Attackers can exhaust server CPU and memory resources by injecting a specific template, rendering the application unresponsive to legitimate requests. This excessive consumption can escalate to a complete system crash (OOM) of the underlying host.

## Reference

* [Project Repository](https://github.com/dgtlmoon/changedetection.io)

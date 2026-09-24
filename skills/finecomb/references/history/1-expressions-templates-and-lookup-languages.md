# 1 Expressions, templates and lookup languages

| Pattern | Historical cases | What to look for | Where it lands |
| --- | --- | --- | --- |
| **Logging and string interpolation interpret lookup expressions** | Log4Shell (CVE-2021-44228, `${jndi:...}` in a log message triggers remote loading); Text4Shell (CVE-2022-42889) | Whether logging, formatting or string substitution components interpret variables or lookup syntax in the message; whether external input can reach these places | [4.13](../specialties/4.13-logs-metrics-and-tracing.md), [4.22](../specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md) |
| **Request fields evaluated as expressions** | Struts (CVE-2017-5638, OGNL in the Content-Type header); Confluence (CVE-2022-26134, CVE-2021-26084, OGNL); Spring Cloud Function (CVE-2022-22963, SpEL in a routing header) | Whether request headers, parameters or routing configuration reach evaluation in an expression language (OGNL, SpEL, EL, MVEL and others) | [27](../dimensions/27-security-and-trust-boundaries.md), [4.22](../specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md) |
| User content treated as a template | Server-side template injection (SSTI) in many frameworks | Whether a user-controlled string is used as template source rather than template data | [4.17](../specialties/4.17-templates-and-text-output.md) |

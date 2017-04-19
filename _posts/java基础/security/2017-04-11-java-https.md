---
title: java 证书
tags: [security]
---

If SpagoBI Server is communicating in HTTPS (esample: https://myserver:8443/SpagoBI) with authentication on its internal repository, with a self-signed certificate, retrieve the certificate itself (you can use the web browser to do this) and import it into your JDK certificates' repository (the cacerts file): launch the command:

JAVA_HOME/bin/keytool

```
    -import
    -trustcacerts
    -alias <provide an alias>
    -file <certificate file>
    -keystore JAVA_HOME/jre/lib/security/cacerts
```
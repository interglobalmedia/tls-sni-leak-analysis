# Overview

This analysis points out that even though TLS packets are encrypted over HTTPS, there is a point in time when a leak occurs. I use the term intentionally, not to pinpoint a weakness in the process. This analysis describes the steps to TLS packet encryption and decryption, and the moment when encryption has not yet taken place, resulting in a packet "leak".

---

[← Back to index](README.md)

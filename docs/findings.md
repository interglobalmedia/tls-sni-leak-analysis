# Findings

## Filtering for and looking at a TLS Client Hello packet directly to see the Server Name Indication field itself in plaintext

The Application Data packet I decrypted was a TLS Client Hello packet with an SNI field which rendered in plaintext when it was decrypted.

There is a leak in the TLS transmission. It is the Server Name Indication (SNI) field inside the TLS Client Hello message. Even though TLS encrypts the content of a connection, the very first message the client sends happens before any encryption keys exist, so it's transmitted entirely in plaintext. Because many websites share the same server IP address, the client has to tell the server which hostname it wants before the handshake can proceed. That hostname goes into the SNI field. Since this message isn't encrypted, anyone capturing traffic on the network can see exactly which site I'm connecting to, even though everything that follows (the actual page content) is encrypted.

## What is a TLS handshake?

`205, "Client Hello (SNI=mar..."` is a TLS handshake packet.

A TLS handshake is crucial for establishing a secure connection between a client (like a web browser) and a server (like a website). This process ensures that data transmitted over the internet is encrypted and secure from eavesdropping.

## Steps to the TLS Handshake (version 1.3)

The TLS handshake highlighted here is version 1.3 (visible in the Protocol column in the screenshot).

| Step | What is included |
| --- | --- |
| **Key Exchange** | <ul>Establishes shared keying material.</ul><ul>Determines the cryptographic parameters, including:<li>Named groups for shared keys (e.g., ECDHE or DHE).</li><li>Symmetric cipher options for encryption.</li></ul> |
| **Server Parameters** | <ul><li>Sets additional handshake parameters.</li><li>Determines if certificate-based client authentication is required.</li></ul> |
| **Authentication** | <ul>Authenticates the server and optionally the client.</ul><ul>Confirms the integrity of the handshake and the keys exchanged.</ul> |

## Steps in the TLS Handshake (version 1.2)

| Step | What is included |
| --- | --- |
| ClientHello | The client sends a "ClientHello" message to the server. |
| The message includes: | Supported TLS versions, list of cipher suites (encryption methods), a random byte string known as "client_random" |
| ServerHello | The server responds with a "ServerHello" message. |
| This message contains: | The chosen TLS version, the selected cipher suite, its own random byte string known as "server_random" |
|  Server Authentication | The server sends its digital certificate to the client. |
|| The client verifies the server's identity using this certificate. |
| Key Exchange | The client generates a session key using the server's public key and sends it to the server. |
|| This key is used for symmetric encryption during the session. |
| Finished Messages | The client sends a "Finished" message encrypted with the session key. |
|| The server responds with its own "Finished" message, also encrypted. |

My `ClientHello` step maps to packet `205 (15.183826)`, the one showing `Client Hello (SNI=mariadcampbell.com...)`. This is the packet that demonstrates a leak. The SNI field is in plaintext before any encryption is established.

My `ServerHello` step maps to `packet 208 (15.214608)`: `ServerHello`, `Change Cipher Spec`.

My `Server Authentication` step maps to `packet 209 (15.214608)`: Certificate, Certificate Verify, the server presents its cert chain.

Since my capture is TLS 1.3 (visible in the Protocol column), it streamlines the classic handshake (version 1.2) flow my table describes. Key exchange isn't a separate step sent after `ServerHello`. It happens via key material embedded directly in the `ClientHello` and` ServerHello` themselves (a "key_share" extension), and the `certificate exchange` (packet 209) is actually encrypted in TLS 1.3, unlike TLS 1.2 where it travels unencrypted.

## TLS handshake 1.2 vs 1.3

| Feature | TLS 1.2 | TLS 1.3 |
| --- | --- | --- |
| **Round Trips Required** | 2 | 1 |
| **Cipher Suites** | Supports weak algorithms | Only strong cipher suites |
| **Forward secrecy**[^4] | Not mandatory | Mandatory |
| **Renegotiation**[^5] | Present | Removed |

The improvements in TLS 1.3 make it a preferred choice for secure communications, enhancing both performance and security for users.

## The purpose of the TLS handshake

The primary goals of the TLS handshake are:

- Authenticate the server (and optionally the client)
- Agree on encryption standards
- Establish a secure channel for data transmission

Once the handshake is complete, both parties can communicate securely using the established session keys. This process is crucial for protecting sensitive information, such as passwords and credit card details, during online transactions.

---

[^4]: Forward secrecy in TLS ensures that session keys are unique and not compromised even if long-term keys are exposed. This means that past communications remain secure even if a server's private key is later compromised.

[^5]: TLS renegotiation is a process that allows a client and server to establish new cryptographic parameters for an existing TLS connection. However, it has vulnerabilities that can be exploited, such as allowing an attacker to inject malicious content into the connection if not properly secured.

---

[← Back to index](README.md)

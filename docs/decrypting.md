# Decrypting

## Decrypt encrypted traffic

To decrypt the HTTPS traffic in a capture, Wireshark needs the TLS session keys logged at the time the traffic was generated. This means it only works for a capture taken while `SSLKEYLOGFILE` was active. Since my existing capture.pcap was taken without that variable set, it can't be retroactively decrypted. I'd need a fresh capture for this specific demonstration.

First, in a terminal Session, I set the `SSLKEYLOGFILE` environment variable and launch the browser from that same terminal so it inherits it:

```shell
# run one line at a time, and in the below sequence
export SSLKEYLOGFILE=~/sslkeylog.log
# running chromium & ensures that chromium runs in the background so that I can run other commands in the same terminal session/instance
chromium &
# after opening chromium, run tcpdump in a separate terminal instance
sudo tcpdump -i eth0 -s 65535 -w new-decrypted-capture.pcap
# then in same terminal session/window as chromium &, I ran to see if ~/sslkeylog.log appeared in the home directory:
ls -la ~/sslkeylog.log
-rw------- 1 maria maria 25372 Aug 16 00:58 /home/maria/sslkeylog.log
```

The `sslkeylog.log` did actually exist in the home directory.

Chromium recognizes this variable if it's set before the browser is launched. Launching Chromium in a Terminal window other than from the specific one where I exported the `SSLKEYLOGFILE` environment variable wouldn't work since it wouldn't inherit the specific terminal's environment. That's why it was important to run `chromium &`, so that I could run other commands in the same session afterwards.

I had to browse to a site or two as before, and then stop the capture as before.

Then, in Wireshark, I went to `Edit → Preferences → Protocols → TLS`, and in "(Pre)-Master-Secret log filename," I browsed to `~/sslkeylog.log`.

Next I selected `sslkeylog.log`, and then clicked the "Ok" button.

![Screenshot of selecting ~/sslkeylog.log in Preferences -> Protocols -> TLS](../images/Screenshot-2026-08-16-at-1.17.27-AM.jpg)

_Selecting ~/sslkeylog.log in Preferences → Protocols → TLS_

Next. I selected a TLS packet from the packet list by typing tls (lowercase) in the filter bar.
 `→ Follow → TLS Stream`, Wireshark automatically decrypted matching TLS sessions in the capture. I saw it reflected as a new "Decrypted TLS" tab in a stream, or plaintext HTTP data instead of raw TLS Application Data records when I followed the stream.

![Screenshot of filtering TLS streams residing in the sslkeylog.log file](../images/Screenshot-2026-08-16-at-1.23.03-AM.jpg)

_Filtering TLS streams residing in the sslkeylog.log file_

The TLS stream I selected to decrypt was the one that starts with `205 15.183826`. When I right-clicked it, I selected `Follow → TLS Stream`. When I did that, the following appeared:

![Screenshot of decrypted https traffic](../images/Screenshot-2026-08-16-at-1.27.06-AM.jpg)

_Decrypted https traffic_

To prove decryption actually worked, an Application Data packet (as I selected) is the best choice. It's the one carrying the real page content, so `Follow → TLS Stream` on it would show me the actual `HTTP request/response` (URLs, headers, maybe cookies) instead of the `raw ciphertext bytes` I'd otherwise see.

```html
<!doctype html>
<!-- https://vercel.app -->
<h1>Redirecting (307)</h1>
The document has moved
<a href="https://www.mariadcampbell.com/">here</a>
```

This is a 307 redirect response from Vercel, sending mariadcampbell.com to www.mariadcampbell.com: plaintext HTML that would otherwise have been fully hidden inside the encrypted TLS traffic.

---

[← Back to index](README.md)

<!-- cSpell:ignore Apache -->
# TLS configuration and hardening

Transport Layer Security (TLS, [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446))
encrypts and authenticates the network connections to Ironic's services. Ironic
in Metal3 terminates TLS on more than one TCP port (see
[RFC 6335](https://www.rfc-editor.org/rfc/rfc6335) for the meaning of "port").
The Ironic Standalone Operator (IrSO) controls whether TLS is enabled and which
certificates are used, while the protocol, cipher and curve tuning lives in the
ironic-image container environment. This page brings both together: which ports
exist, how to enable TLS on each, the flags that tune the protocol versions,
cipher suites and curves, and a recommended hardened baseline.

Running Ironic without TLS is not recommended. Enabling it is covered below and
in [Install Ironic with IrSO](../irso/install-basics.md); supplying and rotating
the certificates themselves is a separate concern from the cipher/protocol
tuning described here.

## The TLS ports

A Metal3 Ironic deployment exposes several externally reachable TLS ports, each
served by Apache with its own certificate and its own set of tuning flags:

- **Ironic API (TCP `6385`).** The REST API that Bare Metal Operator (BMO) and
  administrators talk to. Apache terminates TLS on this port and proxies
  requests to the Ironic API server over a pod-local unix socket
  (`/shared/ironic.sock`), so there is no internal TCP connection to Ironic. The
  exception is the `/images` path, which Apache serves directly from local
  storage rather than proxying — this is how the OS/node images that the Ironic
  Python Agent (IPA) downloads during provisioning are hosted, over the same
  port and certificate. Clients of this port are BMO, the IPA running in the
  ramdisk, and any operator tooling.
- **Virtual-media image server (TCP `6183` for HTTPS, `6180` for plain HTTP).**
  The HTTP server that presents boot and configuration images to a host's BMC
  when provisioning over virtual media. It has its own certificate slot and its
  own tuning flags, though under IrSO it is populated with the same certificate
  as the API; its clients are the BMCs themselves, whose TLS support varies
  widely between vendors and firmware versions.

Because the clients differ, the ports are tuned independently. The API port
faces software you control (BMO, IPA) and can be hardened aggressively. The
virtual-media port faces BMC firmware you often do not control, so overly strict
settings there can break provisioning on older hardware.

Network boot (iPXE) over HTTPS is a third TLS port with its own certificate and
flags. It is out of scope for this page and covered with the network-boot
material instead.

## Enabling TLS

With IrSO, TLS is enabled by referencing a certificate secret from the `Ironic`
resource:

```yaml
spec:
  tls:
    certificateName: ironic-tls
```

The certificate must include the DNS name derived from the service (for example
`ironic.test-ironic.svc`) and the IP address the hosts use to reach Ironic, as a
Subject Alternative Name. See
[Install Ironic with IrSO](../irso/install-basics.md) for a complete self-signed
test example, including generating the secret and its required label. In
production, use a certificate manager such as
[cert-manager](https://cert-manager.io/) to issue and rotate the certificate.

IrSO also enables TLS on the virtual-media image server by default, reusing the
same certificate you supply via `spec.tls.certificateName` rather than a
separately managed one; set `spec.tls.disableVirtualMediaTLS` to turn it off for
BMCs that cannot use HTTPS.

The hardening flags below are environment variables consumed by the
ironic-image container's Apache configuration, not Ironic (`ironic.conf`)
options. IrSO manages the certificates through `spec.tls` but does **not**
currently expose these protocol, cipher and curve flags as dedicated fields,
and `spec.extraConfig` only sets `ironic.conf` options, which is a different
layer. As a result, an IrSO-managed deployment runs with the ironic-image
defaults listed below unless you inject the environment variables yourself by
replacing the `httpd` container through the experimental `spec.overrides`
field. Deployments that build the Ironic pod's environment directly (for
example the classic script-based install) can set them as additional `IRONIC_*`
environment variables alongside the others.

## API TLS flags

These environment variables tune the Ironic API port (and the HTTPS image
server that shares its certificate):

| Variable | Purpose | Default |
|---|---|---|
| `IRONIC_SSL_PROTOCOL` | Allowed TLS protocol versions (Apache `SSLProtocol` directive) | `-ALL +TLSv1.2 +TLSv1.3` |
| `IRONIC_TLS_12_CIPHERS` | Ordered cipher list for TLS 1.2 and below, in OpenSSL format | OpenSSL default |
| `IRONIC_TLS_13_CIPHERS` | Ordered cipher list for TLS 1.3, in OpenSSL format | OpenSSL default |
| `IRONIC_TLS_CURVES` | Ordered list of allowed groups/curves, in OpenSSL format | OpenSSL default |
| `IRONIC_TLS_ENFORCE_SERVER_CIPHER_ORDER` | Make the server, not the client, choose the cipher (TLS 1.2 and below) | `false` |

## Virtual-media TLS flags

These environment variables tune the virtual-media HTTP server. They mirror the
API flags but apply to the separate virtual-media server (which, under IrSO, uses
the same certificate as the API):

| Variable | Purpose | Default |
|---|---|---|
| `IRONIC_VMEDIA_SSL_PROTOCOL` | Allowed TLS protocol versions (Apache `SSLProtocol` directive) | `ALL` |
| `IRONIC_VMEDIA_TLS_12_CIPHERS` | Ordered cipher list for TLS 1.2 and below, in OpenSSL format | OpenSSL default |
| `IRONIC_VMEDIA_TLS_13_CIPHERS` | Ordered cipher list for TLS 1.3, in OpenSSL format | OpenSSL default |
| `IRONIC_VMEDIA_CURVES` | Ordered list of allowed groups/curves, in OpenSSL format | OpenSSL default |
| `IRONIC_VMEDIA_TLS_ENFORCE_SERVER_CIPHER_ORDER` | Make the server choose the cipher (TLS 1.2 and below) | `false` |

Note that `IRONIC_VMEDIA_SSL_PROTOCOL` defaults to `ALL`, which permits every
protocol version the underlying OpenSSL build supports, including legacy ones.
This is deliberately permissive because BMC TLS stacks are often old, but it
means the virtual-media port is *not* hardened by default. Tightening it is
worthwhile, but do so carefully and test against your hardware (see the caveats
below).

## Recommended hardened baseline

The following is a reasonable modern baseline that keeps TLS 1.2 for
compatibility, restricts cipher suites to forward-secret authenticated-encryption
suites, and lets the server dictate the ordering. Cipher and curve strings use
the OpenSSL format.

For the API port:

```bash
IRONIC_SSL_PROTOCOL="-ALL +TLSv1.2 +TLSv1.3"
IRONIC_TLS_13_CIPHERS="TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256"
IRONIC_TLS_12_CIPHERS="ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
IRONIC_TLS_CURVES="x25519:secp256r1:secp384r1"
IRONIC_TLS_ENFORCE_SERVER_CIPHER_ORDER="true"
```

For the virtual-media port, start from the same protocol, cipher, curve and
ordering values, but treat them as a starting point and test against your BMCs:

```bash
IRONIC_VMEDIA_SSL_PROTOCOL="-ALL +TLSv1.2 +TLSv1.3"
IRONIC_VMEDIA_TLS_13_CIPHERS="TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256"
IRONIC_VMEDIA_TLS_12_CIPHERS="ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
IRONIC_VMEDIA_CURVES="x25519:secp256r1:secp384r1"
IRONIC_VMEDIA_TLS_ENFORCE_SERVER_CIPHER_ORDER="true"
```

Notes and caveats:

- **Test virtual-media hardening against your BMCs.** BMC TLS stacks are often
  older than the software clients of the API. If provisioning over virtual media
  fails after tightening these values, relax the virtual-media cipher list (or
  keep TLS 1.2 suites) while leaving the API port strict.
- **TLS 1.3 has no server-side ordering knob.** The
  `*_ENFORCE_SERVER_CIPHER_ORDER` flags apply to TLS 1.2 and below; TLS 1.3
  cipher selection is governed by the client per the protocol.
- **TLS 1.3-only is rarely safe here.** The IPA is a client of the API port and
  cannot natively be restricted to TLS 1.3 only (only custom IPA builds can,
  depending on the host OS and software stack), so keep TLS 1.2 enabled on the
  API port. iPXE has no TLS 1.3 support at all — it negotiates up to TLS 1.2 —
  so the iPXE port cannot be TLS 1.3-only either. Restrict a port to TLS 1.3
  (`-ALL +TLSv1.3`) only where every client of that port is known to support it.
- **Validate the result** with a tool such as `openssl s_client` or a TLS
  scanner against port `6385` (Ironic API, which also serves node images) and
  port `6183` (virtual-media TLS server) after applying changes. Port `6180` is
  the plain-HTTP virtual-media listener and has no TLS to scan.

# https-connection-test

A step-by-step guide to verify that your system can reach a domain service over a secure HTTPS connection.

## ✅ Example Target

Test URL: `https://test-domain.com`

---

## Step 1: Confirm Network-Level Reachability

Ensure the host is reachable via TCP port 443 (HTTPS):

```sh
telnet test-domain.com 443
```

If the connection is refused or times out, troubleshoot networking or firewall settings before proceeding.

## Step 2: Test HTTPS and SSL Behavior with curl

Use curl to examine the HTTPS connection and detect issues like legacy SSL renegotiation or certificate trust errors:

 ```sh
 curl -v https://test-domain.com
 ```

### Possible Outcome - Legacy SSL Renegotiation Detected

```msg
* curl: (35) error:0A000152:SSL routines::unsafe legacy renegotiation disabled
```
This means the server uses legacy SSL renegotiation, and your client (running OpenSSL 3.x) blocks it by default.

Solutions:

- Option 1: Use OpenSSL 1.1.x (Legacy-Compatible)

   Use an older Docker image that bundles OpenSSL 1.1.x, such as:

   ```sh 
   docker run --rm curlimages/curl:7.78.0 curl -v https://test-domain.com
   ```

- Option 2: Enable Unsafe Legacy Renegotiation in OpenSSL 3.x

   Create a custom OpenSSL config file named `openssl_legacy.cnf`:
   ```conf
   # openssl_legacy.cnf
   openssl_conf = openssl_init
   
   [openssl_init]
   ssl_conf = ssl_sect
   
   [ssl_sect]
   system_default = ssl_default_sect
   
   [ssl_default_sect]
   Options = UnsafeLegacyRenegotiation
   ```
   
   Then run curl using this config:
   ```sh
   OPENSSL_CONF=./openssl_legacy.cnf curl -v https://test-domain.com
   ```

### Possible Outcome - CA Certificate Trust Error

```msg
curl: (60) SSL certificate problem: unable to get local issuer certificate
```

Solutions: Verify HTTPS Connection with CA Certificate

  To resolve trust issues when using `curl` with HTTPS:
    
  - Ensure you have the **full certificate chain** in PEM format.
    - This should include the server certificate and all intermediate certificates.
  - Use the `--cacert` option to explicitly tell `curl` which CA certificate to trust.

  ```sh
  curl --cacert /path/to/full-chain.pem https://test-domain.com
  ```

## Summary

To reach https://test-domain.com from your system:

- Enable **legacy SSL renegotiation** if your environment uses OpenSSL 3.x.
- Import and trust the correct **CA certificate chain**.





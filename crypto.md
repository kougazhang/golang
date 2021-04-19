## 不支持 certificate CN

... until the site's certificate is updated to use SANs. This is not an issue with Singularity and Harbor, but with the certificate in use for ngc.em.io. Singularity is written in Go, and Go is deprecating support for certificate CN matching:

https://golang.org/pkg/crypto/x509/#Certificate.VerifyHostname

> The legacy Common Name field is ignored unless it's a valid hostname, the certificate doesn't have any Subject Alternative Names, and the GODEBUG environment variable is set to "x509ignoreCN=0". Support for Common Name is deprecated will be entirely removed in the future.

解决方案: 设置 `export GODEBUG=x509ignoreCN=0`


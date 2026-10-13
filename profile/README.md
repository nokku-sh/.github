# Nokku

Short-lived SSH certificates for your servers, with plain OpenSSH on the client.

Everything lives in [nokku](https://github.com/nokku-sh/nokku).

- [nokku](https://github.com/nokku-sh/nokku/tree/main/core) is the core. It decides who may log in where and signs the certificates.
- [nokkud](https://github.com/nokku-sh/nokku/tree/main/daemon) is the daemon for your servers. It renews host certificates and enforces access policies.
- [nk](https://github.com/nokku-sh/nokku/tree/main/cli) is the CLI. It signs you in and keeps your certificates fresh.
- [proto](https://github.com/nokku-sh/nokku/tree/main/proto) is the API schema.

Found a vulnerability? Please use private vulnerability reporting in the Security tab of the [nokku](https://github.com/nokku-sh/nokku) repo.

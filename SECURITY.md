# Security

## Reporting a vulnerability

Please do not open a public issue for a suspected vulnerability. Use GitHub's private reporting instead:

1. Open the repository's **Security and quality** tab.
2. Select **Report a vulnerability**.
3. Include as much of this as you can:
   - Which version, and the platform it runs on
   - What the issue is and what an attacker gains
   - Steps to reproduce, or a minimal proof of concept
   - Logs that help, with secrets and tokens removed

Reports are handled confidentially. If you would like credit in the advisory or the changelog, say so.

Nokku is built and maintained by one person, so this is best effort:

- **A reply**, usually within a week
- **Serious issues first**
- **A fix or clear guidance** as soon as possible, depending on complexity and impact

Only the latest release gets security fixes. Older releases are not patched, so upgrade to the newest one.

## Verifying a release

Releases are signed with cosign by the shared release workflow, so every repo has the same certificate identity. The repository flag ties a release to its repo, always pass it:

```sh
cosign verify-blob --bundle checksums.txt.sigstore.json \
  --certificate-identity https://github.com/nokku-sh/.github/.github/workflows/go-release.yaml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-github-workflow-repository nokku-sh/<repo> \
  checksums.txt
```

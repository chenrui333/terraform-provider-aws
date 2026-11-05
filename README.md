# jw-release

Release branch for Justworks canary builds of `terraform-provider-aws`. It holds no provider
source. The `jw-release` workflow builds the commit named in `jw-release.json`, then signs and
publishes it as a GitHub pre-release. It is not an official HashiCorp release.

## Cut a canary

1. Push the patched commit to any branch of this fork.
2. Edit `jw-release.json`. Set `ref` to the full commit SHA and `version` to a new version
   (for example `6.73.0-jw.2`). The workflow refuses to overwrite an existing release.
3. Commit and push to `jw-release`, or run the workflow with `workflow_dispatch`.

The workflow builds `linux_amd64`, `linux_arm64` and `darwin_arm64` with
`CGO_ENABLED=0 go build -trimpath -buildvcs=false -ldflags "-s -w -X …ProviderVersion=<version>"`,
using the Go version from the ref's `go.mod`. It writes the registry manifest and `SHA256SUMS`, and
signs `SHA256SUMS` with key `BC8C30954D11DF6BAB41BE730BB9E23BABAF1127` from the secret
`JW_CANARY_GPG_PRIVATE_KEY`. It then creates the annotated tag `v<version>` at `ref` and a
pre-release that is not marked latest, with `PROVENANCE.md` attached.

## Verify a release

```sh
gpg --import terraform-provider-aws_<version>_signing-key.pub.asc
gpg --verify terraform-provider-aws_<version>_SHA256SUMS.sig terraform-provider-aws_<version>_SHA256SUMS
sha256sum -c terraform-provider-aws_<version>_SHA256SUMS
```

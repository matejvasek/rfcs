# Stack Descriptor and Tooling

## Summary

The stack-building codebase,
[`create-stack`](https://github.com/paketo-buildpacks/stacks/tree/main/create-stack)
should be moved into the `jam` tooling CLI, and allow users to configure their
stack build via a common configuration file.

## Motivation

Today, `create-stack` is highly coupled to the `bionic` stacks we currently
support. This makes it difficult to support alternative stacks and platforms or
to allow the codebase to be reused by external parties for developing their own
stack.

## Detailed Explanation

The `create-stack` codebase will be refactored to allow users to build stacks
using a configuration file with the following format:

```toml
id = "<stack id>"                # required
homepage = "<homepage uri>"      # optional
maintainer = "<maintainer name>" # optional

platforms = ["linux/amd64", "linux/arm64", "<other-platform>"] # optional; default = ["linux/amd64"]; choices defined in https://docs.docker.com/desktop/multi-arch/

[distro]
  name = "<os distro name>"       # optional
  version = "os distro version>"  # optional

[build]
  tag = "<image tag>"                 # required
  dockerfile = "<path to dockerfile>" # required
  description = "<image description>" # optional

  uid = 1001                   # required
  gid = 1000                   # required
  shell = "<user login shell>" # optional; default = "/sbin/nologin"

  [build.args] # optional
    example-arg = "<example-value>"

[run]
  tag = "<image tag>"                 # required
  dockerfile = "<path to dockerfile>" # required
  description = "<image description>" # optional

  uid = 1002                   # required
  gid = 1000                   # required
  shell = "<user login shell>" # optional; default = "/sbin/nologin"

  [run.args] # optional
    example-arg = "<example-value>"

[deprecated]
  legacy-sbom = true # optional; default = false; enables https://github.com/paketo-buildpacks/rfcs/blob/main/text/stacks/0001-stack-package-metadata.md
  mixins = true      # optional; default = false; enables the inclusion of the `io.buildpacks.stack.mixins` label
```

Then the codebase will be merged into the `jam` CLI under a `create-stack`
command. Once complete, a user should be able to build a stack with a given
descriptor file, `stack.toml`, using the following command:

`jam create-stack --config stack.toml`

## Rationale and Alternatives

We could abandon the `create-stack` codebase entirely and expect that anyone
developing stacks just build a `Dockerfile` following the
[directions](https://buildpacks.io/docs/operator-guide/create-a-stack/)
outlined in the buildpacks documentation.

## Implementation

The `create-stack` command will be refactored to support this new configuration
file format and the features it allows. The codebase will then be moved into
the `jam` codebase as a command within that tool. This aligns more with how the
project maintains its tooling and allows us to keep tooling consolidated under
the tooling subteam.

## Prior Art

The buildpacks spec defines several descriptor files in TOML format.

[TOC]
# Developer's Guide 
## How can I help ?

You can join Nocalhost's [discord](https://discord.gg/Fe4ExSNwJd) to get involved in our community.

To dig deeper, check out the [architecture design docs](https://istio.io/docs/concepts/what-is-istio/#architecture) 

If you're looking for something to do to get your feet wet working on Nocalhost, look for GitHub issues marked with the Help Wanted label：

- [Primary Nocalhost repo](https://github.com/nocalhost/nocalhost/labels/help%20wanted)
- [Documentation repo](https://github.com/nocalhost/nocalhost-web/labels/help%20wanted)

Of course, even if there's not an issue opened for it, you can always contribute more docs or blogs. 

## Using the Code Base

This document helps you get started using the Nocalhost code base. If you follow this guide and find some problem, please take a few minutes to update this page.

### Cloning the code

Clone the main `nocalhost` repo :

```shell
git clone https://github.com/nocalhost/nocalhost.git
cd nocalhost
```

### Building the code

To build nhctl for Intel Silicon

```shell
make nhctl
```

Or build nhctl for Mac M1 Silicon

```shell
make nhctl-osx-arm64
```

### Copy binary to IDE plugin work directory

VSCode and Jetbrains plugins use `nhctl` in  `~/.nh/bin`. if you want your developing version used by plugins, you need to copy `nhctl` to that directory. Take `Mac M1` for example:

```shell
cp build/nhctl-darwin-arm64 ~/.nh/bin/nhctl
```



## Pull requests

If you're working on an existing issue, simply respond to the issue and express interest in working on it. This helps other people know that the issue is active, and hopefully prevents duplicated efforts.

To submit a proposed change:

- Fork the repository.

- Create a new branch for your changes.

- Develop the code/fix.

- Modify the documentation as necessary.

- Create a PR to the `dev` branch of `nocalhost` repository.

  

PRs may only be merged after the following criteria are met:

1. It has no `NO LGTM` comment from a reviewer.
2. It has been `LGTM`-ed by at least one of the maintainers of that repository.
3. All Github actions related have passed.

## Nocalhost Community Code of Conduct

Nocalhost follows the [CNCF Code of Conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md).

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported by contacting
the [Kubernetes Code of Conduct Committee](./committee-code-of-conduct) via <conduct@kubernetes.io>.
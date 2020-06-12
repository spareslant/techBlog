---
title: "Bosh"
date: 2020-05-03T11:02:46+01:00
draft: true
---

## Bosh concept

* A package is a particular software i.e curl, zookeeper etc
* A release may comprise of a multiple packages.
* A deployment consists of instructions about how to deploy a release.
* In BOSH world, a BOSH-Release is actually a bundle of your source code and dependent files. A BOSH-Release is never a compiled release of your source code. This is important distinction because in normal world, a release means, it already has compiled binaries. When you actually deploy a BOSH-Release, at that time, your source code is compiled. And at the same time your compiled source also gets deployed. The instructions about how to compile source-code and how-to deploy it are the part of BOSH-Release.

## release creation process:
* create src code
   * src code package creation and note down dependencies.
* add blobs (dependencies in tar.gz) if required.
   * package creation for each blob
* create jobs


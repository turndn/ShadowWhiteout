# ShadowWhiteout capable Overlay File Systems

The Shadow Whiteout is a concept presented in "Nesting Overlay File Systems with ShadowWhiteout" (APSys 2025).
This repository contains the research prototype of Shadow Whiteout capable overlayfs for Linux 6.7.11.

## Build

```
cd fs/overlayfs
make
```

## Install

```
cd fs/overlayfs
make reload
```

## Branches

* `main`: overlayfs with Shadow Whiteout.
* `shadow-whiteout-regular`: overlayfs with Shadow Whiteout regular file version.

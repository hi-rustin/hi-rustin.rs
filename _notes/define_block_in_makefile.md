---
title: 'How to define a block in a Makefile?'
layout: post
---

## How to define a block in a Makefile?

We can define a block in a Makefile by using the following syntax:

```makefile
define <block_name>
    <commands>
endef
```

At same time, we can define some variables in the block, and use them in the block.

```makefile
define go_build
 GOOS=$(GOOS) GOARCH=$(GOARCH) CGO_ENABLED=0 $(GO) build -ldflags "-extldflags \"-static\" $(1)" ./cmd/a
 GOOS=$(GOOS) GOARCH=$(GOARCH) CGO_ENABLED=0 $(GO) build -ldflags "-extldflags \"-static\" $(1)" ./cmd/b
endef

build:
 $(call go_build,$(GO_LDFLAGS))
```

We pass the `$(GO_LDFLAGS)` to the `go_build` block, and use it in the block.

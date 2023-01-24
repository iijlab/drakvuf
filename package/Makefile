# Makefile for build the Debian packages with Docker

# Settings
## env
SHELL := /bin/sh
TZ := JST-9

# Targets
.DEFAULT_GOAL := help

## [help]
.PHONY: help
help: ## Display available targets
	@printf "\033[01mBuild the Debian packages with Docker\033[0m\n"
	@printf "\033[01m--------\033[0m\n"
	@printf "\033[01m$$ make <target> [T=ubuntu-20.04-amd64]\033[0m\n"
	@printf "\033[01m$$ make deb [T=ubuntu-20.04-amd64]\033[0m\n"
	@printf "\033[01m$$ make sh [T=ubuntu-20.04-amd64]\033[0m\n"
	@printf "\n"
	@printf "\033[32m<target>\033[0m\n"
	@printf "\033[32m--------\033[0m\n"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) \
	| sort \
	| awk 'BEGIN {FS=":.*?## "}; {printf "\033[32m%-20s\033[0m %s\n", $$1, $$2}'
	@printf "\033[32m--------\033[0m\n"

## [docker builder]
### vars
CREATE_DONE := .stamp_docker_builder
### builder
BUILDER_NAME := drakvuf-builder
#### vars/driveropts
ifneq ($(strip $(http_proxy)),)
	DRIVEROPTS += --driver-opt env.http_proxy="$(http_proxy)"
endif
ifneq ($(strip $(https_proxy)),)
	DRIVEROPTS += --driver-opt env.https_proxy="$(https_proxy)"
endif
ifneq ($(strip $(no_proxy)),)
	DRIVEROPTS += --driver-opt \"env.no_proxy="$(no_proxy)"\"
endif

### rules
.PHONY: builder
builder: $(CREATE_DONE) ## Create a new builder instance
$(CREATE_DONE): export DOCKER_BUILDKIT=1
$(CREATE_DONE):
	docker buildx create --name $(BUILDER_NAME) $(DRIVEROPTS)
	touch $@

.PHONY: builder-run
builder-run: $(CREATE_DONE) ## Launch builder container
	docker buildx --builder $(BUILDER_NAME) inspect --bootstrap

.PHONY: builder-clean
builder-clean: $(CREATE_DONE) ## Remove docker builder instance
	docker buildx --builder $(BUILDER_NAME) rm
	rm -f $(CREATE_DONE)

.PHONY: cache
cache: ## Show disk usage for docker builder cache
	@$(MAKE) builder-run >/dev/null
	docker buildx --builder $(BUILDER_NAME) du

.PHONY: cache-clean
cache-clean: ## Remove docker builder cache
	@$(MAKE) builder-run >/dev/null
	docker buildx --builder $(BUILDER_NAME) prune -f

## [docker image]
### vars
TAG := ubuntu-20.04-amd64 # Default
ifdef T
	TAG := $(T)
endif
DISTRIBUTOR := $(word 1, $(subst -, ,$(TAG)))
RELEASE := $(word 2, $(subst -, ,$(TAG)))
ARCH := $(word 3, $(subst -, ,$(TAG)))
BUILD_DONE := .stamp_docker_image.$(TAG)
DOCKERFILE := Dockerfile.$(TAG)
BUILD_TARGET := drakvuf-builder
BUILD_IMAGE := nedo/$(BUILD_TARGET):$(TAG)
#### vars/buildargs
BUILDARGS := --builder $(BUILDER_NAME) --load --platform linux/$(ARCH) --force-rm
ifneq ($(strip $(VERBOSE)),)
	BUILDARGS += --progress=plain
endif
BUILDARGS += --build-arg IMAGE=$(DISTRIBUTOR):$(RELEASE)
BUILDARGS += --build-arg TZ=$(TZ)
ifneq ($(strip $(http_proxy)),)
	BUILDARGS += --build-arg http_proxy=$(http_proxy)
endif
ifneq ($(strip $(https_proxy)),)
	BUILDARGS += --build-arg https_proxy=$(https_proxy)
endif
ifneq ($(strip $(no_proxy)),)
	BUILDARGS += --build-arg no_proxy=$(no_proxy)
endif

### rules
.PHONY: image
image: $(BUILD_DONE) ## Create docker image
$(BUILD_DONE): export DOCKER_BUILDKIT=1
$(BUILD_DONE): $(DOCKERFILE) $(CREATE_DONE)
	docker run --privileged --rm tonistiigi/binfmt --install all
	cd .. && \
	  docker buildx build $(BUILDARGS) -t $(BUILD_IMAGE) -f ./package/$(DOCKERFILE) --target=$(BUILD_TARGET) .
	docker image prune -f
	touch $@

.PHONY: image-clean
image-clean: ## Remove docker image
	docker image rm $(BUILD_IMAGE)
	rm -f $(BUILD_DONE)

.PHONY: image-allclean
image-allclean: ## Remove all docker images
	@docker image ls -f reference='nedo/drakvuf-builder:*' --format "{{.Tag}}" \
	| xargs -I{} -n 1 -t $(MAKE) image-clean T={}

## [docker run]
### vars
#### vars/runargs
RUNARGS := -e TZ=$(TZ)
ifneq ($(strip $(http_proxy)),)
	RUNARGS += -e http_proxy=$(http_proxy)
endif
ifneq ($(strip $(https_proxy)),)
	RUNARGS += -e https_proxy=$(https_proxy)
endif
ifneq ($(strip $(no_proxy)),)
	RUNARGS += -e no_proxy=$(no_proxy)
endif
#### vars/mount
BASEDIR := $(shell pwd)/out
DISTDIR := $(BASEDIR)/$(DISTRIBUTOR)/$(RELEASE)/$(ARCH)

### functions
define docker-run
docker container run $1 --rm \
 --mount type=bind,src=$(DISTDIR),dst=/out,consistency=cached \
 $(BUILD_IMAGE)
endef

### rules
$(DISTDIR):
	mkdir -pv $(DISTDIR)

.PHONY: deb
deb: $(DISTDIR) image ## Create the debian packages
	$(call docker-run)

.PHONY: sh
sh: RUNARGS += -it --entrypoint=bash
sh: $(DISTDIR) image ## Enter interactive mode and start shell in container
	$(call docker-run,$(RUNARGS))

## [clean]
### rules
.PHONY: realclean
realclean: ## Remove the all debian packages, docker images and builder instance
	rm -rfv $(BASEDIR)
	-$(MAKE) image-allclean
	-$(MAKE) builder-clean

.PHONY: distclean
distclean: clean ## Remove the debian packages and docker image
	-$(MAKE) image-clean

.PHONY: clean
clean: ## Remove the debian packages
	rm -rfv $(DISTDIR)

# Makefile ends here

# SPDX-license-identifier: Apache-2.0
##############################################################################
# Copyright (c) 2019 Intel Corporation
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
##############################################################################

# Common
VERSION := $(shell git describe --tags --always)
BUILD := $(shell git rev-parse --short HEAD)
PROJECTNAME := $(shell basename "$(PWD)")

ROOTPATH := $(shell realpath "$(PWD)/../")
COP := ${PWD}/build/_output/bin/collectd-operator
COP_LOCAL := ${COP}-local
ifndef IMAGE_NAME
override IMAGE_NAME := dcr.cluster.local:32644/collectd-operator:latest
endif

export GO111MODULE=on

.PHONY: clean plugins

## build: Generate the k8s and openapi artifacts using operator-sdk
build: clean format
	GOOS=linux GOARCH=amd64
	operator-sdk generate k8s --verbose
	operator-sdk generate openapi --verbose
	@go build -o ${COP_LOCAL} collectd-operator/cmd/manager
	@operator-sdk build ${IMAGE_NAME} --verbose

# The following is done this way as each patch on CI runs build and each merge runs deploy. So for build we don't need to build binary and hence
# no need to create a static binary with additional flags. However, for generating binary, additional build flags are necessary. This if used with
# mock plugin errors out for unit tests. So the seperation avoids the error.

## all: Delete the image, binary, complete build, test and run coverage
all: build test cover
## debug: Build local binary for debugging

debug:
	@echo "Deleting ${COP_LOCAL} binary"
	@rm -rf ${COP_LOCAL}
	operator-sdk generate k8s --verbose
	operator-sdk generate openapi --verbose
	@echo "Building ${COP_LOCAL} binary"
	@go build -o ${COP_LOCAL} collectd-operator/cmd/manager

## deploy: Build Dockerfile and publish to repository
deploy: build test publish

## publish: Push docker image to repository
publish:
	@docker push ${IMAGE_NAME}
.PHONY: test
## test: run tests
test: clean
	@go test -v ./...
## format: format source code
format:
	@go fmt ./...

## clean: clean build artifacts, image, binary 
clean:
	@echo "Deleting the collectd-operator binary"
	@rm -rf ${COP}
	@rm -rf ${COP_LOCAL}
	@echo "Deleting the collectd-operator docker image"
	@docker 2>/dev/null rmi ${IMAGE_NAME} | true

.PHONY: cover
## cover: run tests and generate coverage report
cover:
	@go test ./... -coverprofile=coverage.out
	@go tool cover -html=coverage.out -o coverage.html

.PHONY: help
## help: Print help message
help: Makefile
	@echo
	@echo " Requires Operator SDK v0.8.0+, go1.12.5+"
	@echo
	@echo " Choose a command run in "$(PROJECTNAME)":"
	@echo
	@sed -n 's/^## //p' $< | column -t -s ':' |  sed -e 's/^/ /'
	@echo

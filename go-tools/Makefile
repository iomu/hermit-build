export CGO_ENABLED = 0
export GOOS ?= $(shell go env GOOS)
export GOARCH ?= $(shell go env GOARCH)

SUFFIX = $(GOOS)-$(GOARCH)

UNVERSIONED_TOOLS = 

TOOLS = github.com/google/gnostic:cmd/protoc-gen-openapi

all:
	make GOOS=linux GOARCH=amd64 clean build
	make GOOS=linux GOARCH=arm64 clean build
	make GOOS=darwin GOARCH=amd64 clean build
	make GOOS=darwin GOARCH=arm64 clean build

build:
	for tool in $(TOOLS); do \
		module=$$(echo $$tool | cut -d: -f1); \
		pkg=$$(echo $$tool | tr : /); \
		rm -f go.mod go.sum; \
		go mod init install; \
		go get $$pkg@latest; \
		version=$$(go list -m -f '{{ .Version }}' $$module); \
		echo $$module-$$version; \
		binary=$$(echo $$pkg | awk -F / '{print $$NF}'); \
		go build -o $$binary-$$version-$(SUFFIX) $$pkg; \
		bzip2 -9 $$binary-$$version-$(SUFFIX); \
	done
	mkdir -p unversioned
	set -x; \
	for tool in $(UNVERSIONED_TOOLS); do \
		echo $$tool; \
		rm -f go.mod go.sum; \
		go mod init install; \
		go get $$tool; \
		tool=$${tool%@*}; \
		go build -o unversioned/$$(basename $$tool)-$(SUFFIX) $$tool; \
		bzip2 -9 unversioned/$$(basename $$tool)-$(SUFFIX); \
	done

clean:
	rm -f *$(SUFFIX) *$(SUFFIX).bz2 unversioned/*$(SUFFIX).bz2

SEMVER ?= 0.0.1

TEST?=$$(go list ./... )
GOFMT_FILES?=$$(find . -name '*.go')
WEBSITE_REPO=github.com/hashicorp/terraform-website
PKG_NAME=yandex
LINT_PACKAGES= ./yandex/... yandex-framework/...

SWEEP?=$(YC_REGION)
ifeq ($(SWEEP),)
SWEEP=ru-central1
endif

SWEEP_DIR= ./yandex ./yandex-framework/test/...

SWEEPERS_FOR_RUNNING?=""

git_version := $(shell git describe --abbrev=0 --tags)
git_hash := $(shell git rev-parse --short HEAD)
current_time = $(shell date +"%Y-%m-%dT%H-%M-%SZ")
LDFLAGS = -ldflags "-s -w -X github.com/yandex-cloud/terraform-provider-yandex/version.ProviderVersion=${git_version}-${current_time}+dev.${git_hash}"


default: build

##build and local-build generate the same user-agent
##The difference is that build will put the file in the $GOPATH/bin folder, while local-build will put it in the tf plugins folder
##Example user-agent: Terraform/1.5.7 (https://www.terraform.io) terraform-provider-yandex/0.124.0-2024-07-26T16-24-43Z+dev.79152e6a grpc-go/1.62.1
build: fmtcheck
	go install ${LDFLAGS}

local-build: fmtcheck
	go build ${LDFLAGS} -o $(HOME)/.terraform.d/plugins/registry.terraform.io/yandex-cloud/yandex/$(SEMVER)/$(shell go env GOOS)_$(shell go env GOARCH)/terraform-provider-yandex main.go

sweep:
	@echo "WARNING: This will destroy infrastructure. Use only in development accounts.";
	go test $(SWEEP_DIR) -v -sweep=$(SWEEP) -sweep-run=$(SWEEPERS_FOR_RUNNING) -timeout 60m

test: fmtcheck
	go test $(TEST) -timeout=30s -parallel=4

testacc: fmtcheck
	TF_ACC=1 TF_SCHEMA_PANIC_ON_ERROR=1 go test $(TEST) -v $(TESTARGS) -timeout 120m

vet:
	@echo "go vet ."
	@go vet $$(go list ./...) ; if [ $$? -eq 1 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

fmt:
	gofmt -w $(GOFMT_FILES)

fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

lint:
	golangci-lint run --modules-download-mode mod $(LINT_PACKAGES)

tools:
	@echo "==> installing required tooling..."
	go install github.com/client9/misspell/cmd/misspell
	go install github.com/golangci/golangci-lint/cmd/golangci-lint

test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./$(PKG_NAME)"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)

website:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

changie-lint:
	go run lint/cmd/changie/changie.go batch patch -d

.PHONY: build sweep test testacc vet fmt fmtcheck lint tools test-compile website changie-lint

.PHONY: all
all:
	docker build -t virtualenv-tools-fpm .
	docker run \
		--env PACKAGE_ITERATION="$(PACKAGE_ITERATION)" \
		--env PACKAGE_MAINTAINER="$(PACKAGE_MAINTAINER)" \
		--env APP_VERSION="$(APP_VERSION)" \
		--tty \
		--rm \
		--volume $(PWD)/..:/build \
	virtualenv-tools-fpm /build/package/package

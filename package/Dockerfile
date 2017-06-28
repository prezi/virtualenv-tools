# Package container with fpm (Linux: alpine) for virtualenv-tools
# Supported workflow:
# - Use fpm for python to .deb packaging

# Why alpine?
# - ruby: => @start ~750MB @end NoData
# - ruby slim: +> @start ~280MB @end NoData
# - debian wheezy => @srat ~85MB @end ~280MB
# - ruby alpine: => @start ~5MB @end ~160MB
# Why NOT alpine, maybe?
# - new Linux flavour, new things to learn

FROM ruby:2.3.1-alpine

# Throw errors if Gemfile has been modified since Gemfile.lock.
RUN bundle config --global frozen 1

COPY Gemfile /
COPY Gemfile.lock /

### FPM installation ###
# Package required for:
# - binutils: provides ar, required by fpm to build deb packages
# - build-base: only for bundle install, remove later
# - python: fpm needs python for python input type
# - py-setuptools: fpm need easy_install for python input type
# - tar: fpm needs GNU tar (busybox's tar does not support some options)
RUN apk add --no-cache \
  bash \
  binutils \
  build-base \
  python \
  py-setuptools \
  tar \
  && bundle install \
  && apk del build-base

# # Add user without password (-D) and with home (-h)
RUN adduser -D -h /build builder
WORKDIR /build
USER builder

CMD ["/bin/bash"]

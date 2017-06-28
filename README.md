# Original readme of virtualenv-tools

-- virtualenv-tools

This repository contains scripts we're using at Fireteam for our
deployment of Python code.  We're using them in combination with
salt to build code on one server on a self contained virtualenv
and then move that over to the destination servers to run.

Why not virtualenv --relocatable?

 For starters: because it does not work.  relocatable is very
 limited in what it does and it works at runtime instead of
 making the whole thing actually move to the new location.  We
 ran into a ton of issues with it and it is currently in the
 process of being phased out.

Why would I want to use it?

 The main reason you want to use this is for build caching.  You
 have one folder where one virtualenv exists, you install the
 latest version of your codebase and all extensions in there, then
 you can make the virtualenv relocate to a target location, put it
 into a tarball, distribute it to all servers and done!

Example flow:

 First time: create the build cache

   $ mkdir /tmp/build-cache
   $ virtualenv --distribute /tmp/build-cache

 Now every time you build:

   $ . /tmp/build-cache/bin/activate
   $ pip install YourApplication

 Build done, package up and copy to whatever location you want to have
 it.

 Once unpacked on the target server, use the virtualenv tools to
 update the paths and make the virtualenv magically work in the new
 location.  For instance we deploy things to a path with the
 hash of the commit in:

   $ virtualenv-tools --update-path /srv/your-application/<hash>

 To also update the Python executable in the virtualenv to the
 system one you can reinitialize it in one go:

   $ virtualenv-tools --reinitialize /srv/your-application/<hash>


Compile once, deploy whereever.  Virtualenvs are completely self
contained.  In order to switch the current version all you need to
do is to relink the builds.

# Prezi specific notes

# Package creation for virtualenv-tools

## How to use it

Define the environment variables `PACKAGE_ITERATION`, `PACKAGE_VERSION` and
`PACKAGE_MAINTAINER`.

- `PACKAGE_ITERATION` is related to
[`debian_revision`](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-Version).
That number starts from 0 and incremented by 1 every time you rebuild a
package with the same application version. If the application version
changes then revision number goes back to 0.
[See next section for detailed help](#choosing-iteration-number)
- `PACKAGE_MAINTAINER` can be just infra@prezi.com
- `APP_VERSION` is the application version of virtualenv-tools. It is a semantic
version number which is on the commit as a git tag (e.g. 1.2). This means that if
you have changed the code, please add the new version number as a git tag.

### Host requirements

- docker
- make

### Backing the `.deb` package

Now you should be ready to call `make` in the `package` directory. What will happen?

1. Make will build an image for packaging, then
2. spins up a container from that image with your `$PWD` mounted.
3. `package` script will build a deb package from `virtualenv-tools`
5. At the end your `.deb` package should be in the `package` current folder.

### Uploading the package

Decide which ubuntu version you would like to deploy for, this defines the codename
you have to use, e.g. `prezi-xenial` or `prezi-precise`.
Copy the file to `oam3` and upload the deb package with the `deb-s3-wrapper-public` script.

***Warning:*** Be careful with the upload, because there is no way to undo it and remove the package. Moreover, if you upload a package with a new iteration number, the same packages with earlier iteration numbers will not be accessible.


Example:
```
deb-s3-wrapper-public upload -b package-repository-public -p --codename=prezi-xenial --arch=amd64 python-virtualenv-tools_1.2-1prezi_all.deb
```

Note: If you would like to upload for multiple ubuntu versions you have to run the upload command multiple times with different codename parameters.

### Changing the version to use in Chef
Check if you have to update the pinned package in the
[virtualenv_tools.rb](https://github.com/prezi/prezi-chef/blob/master/cookbooks/python/recipes/virtualenv_tools.rb) recipe.

Currently we pin the version for `lucid`, `precise` and `trusty`, but not for `xenial`.

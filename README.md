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

## ROLLING DEBIAN PACKAGES:
### Get FPM if necessary:
gem install fpm

### Finding out the new package version
The package version should follow the pattern below:
`<version-number>-<build-number>prezi`

`<version-number>` is a semantic version number which is on the commit as a git tag (e.g. 1.2)

`<build-number>` is just an integer counter starting from 0 and incremented on every build of the same semantic version.

So e.g. the 3rd build of the 1.2 version is `1.2-2prezi`

This means that if you have changed the code, please add the new version number as a git tag.

### Create deb package

From the virtualenv-tools source dir, run the following command with the new package version:
```
fpm -t deb -s python -n python-virtualenv-tools -m infra@prezi.com --prefix /usr/local/ --description "A set of tools for virtualenv" --url "http://github.com/prezi/virtualenv-tools" -v <package-version> --no-python-dependencies --python-bin /usr/bin/python2.7 --python-install-lib lib/python2.6/dist-packages setup.py
```

### Uploading the package

Decide which ubuntu version you would like to deploy for, this defines the codename you have to use, e.g. ubuntu-xenial or ubuntu-precise.
Copy the file to oam3 and upload the deb package (remember to use the actual package version):

```
cp python-virtualenv-tools_<package-version>_all.deb /var/www/packages.prezi.com/pool/main/p/python-virtualenv-tools/`

deb-s3-wrapper upload -b package-repository-public -p --codename=<codename> --arch=amd64 /var/www/packages.prezi.com/pool/main/p/python-virtualenv-tools/python-virtualenv-tools_1.1-0prezi3_all.deb
```

Note: If you would like to upload for multiple ubuntu versions you have to run the upload command multiple times with different codename parameters.

### Changing the version to use in Chef
add action: upgrade to the package def in chef

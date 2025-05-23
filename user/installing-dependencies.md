---
title: Installing Dependencies
layout: en

redirect_from:
  - /user/apt/
---


## Install Packages on Standard Infrastructure

To install Ubuntu packages that are not included in the standard [precise](/user/reference/precise/), [trusty](/user/reference/trusty/), [xenial](/user/reference/xenial/), or [bionic](/user/reference/bionic/) distribution, use apt-get in the `before_install` step of your `.travis.yml`:

```yaml
before_install:
  - sudo apt-get -y install libxml2-dev
```
{: data-file=".travis.yml"}

By default, `apt-get update` does not get run automatically. If you want to update `apt-get update` automatically on every build, there are two ways to do this. The first is by running `apt-get update` explicitly in the `before_install` step:

```yaml
before_install:
  - sudo apt-get update
  - sudo apt-get -y install libxml2-dev
```
{: data-file=".travis.yml"}

The second way is to use the [APT addon](#installing-packages-with-the-apt-addon):

```yaml
before_install:
  - sudo apt-get -y install libxml2-dev
addons:
  apt:
    update: true
```
{: data-file=".travis.yml"}

> Do not run `apt-get upgrade` in your build as it downloads up to 500MB of packages and significantly extends your build time. Additionally, some packages may fail to update, which will lead to a failed build.
>
> Use the `-y` parameter with apt-get to assume yes to all queries by the apt tools.

### Install Packages from a custom APT Repository

For some packages, you may find an existing repository, which isn't yet set up on our build environment by default. You can easily add custom repositories and Launchpad PPAs as part of your build.

For example, to install gcc from the ubuntu-toolchain ppa

```yaml
before_install:
  - sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test
  - sudo apt-get -q update
  - sudo apt-get -y install gcc-4.8
```
{: data-file=".travis.yml"}

For repositories not hosted on Launchpad, you need to add a GnuPG key as well.

If you're installing packages this way, make sure you download the correct version for your environment.

This example adds the APT repository for Varnish 3.0 for Ubuntu 12.04 to the locally available list of APT sources and then installs the `varnish` package.

```yaml
before_script:
  - curl http://repo.varnish-cache.org/debian/GPG-key.txt | sudo apt-key add -
  - echo "deb http://repo.varnish-cache.org/ubuntu/ precise varnish-3.0" | sudo tee -a /etc/apt/sources.list
  - sudo apt-get -qq update
  - sudo apt-get -y install varnish
```
{: data-file=".travis.yml"}

### Install Packages without an APT Repository

For some projects, there may be a Debian/Ubuntu package available, but no corresponding APT repository. These are still easy to install, but require the extra step of downloading.

If you're installing packages this way, make sure you download the correct version for your environment.

Say your project requires the pngquant tool to compress PNG files, here's how to download and install the .deb file:

```yaml
before_install:
  - wget http://pngquant.org/pngquant_1.7.1-1_i386.deb
  - sudo dpkg -i pngquant_1.7.1-1_i386.deb
```
{: data-file=".travis.yml"}

### Install Packages with the APT Addon

You can also install packages and sources using the APT addon, without running `apt-get` commands in your `before_install` script.

If your requirements go beyond the normal installation, please use another method described above.

#### Add APT Sources

To add APT sources, you can use one of the following three types of entries:

1. aliases defined in [source safelist](https://github.com/travis-ci/apt-source-safelist)
2. `sourceline` key-value pairs which will be added to `/etc/apt/sources.list`
3. when APT sources require GPG keys, you can specify this with `key_url` pairs in addition to `sourceline`.

The following snippet shows these three types of APT sources.

```yaml
addons:
  apt:
    sources:
    - deadsnakes
    - sourceline: 'ppa:ubuntu-toolchain-r/test'
    - sourceline: 'deb https://packagecloud.io/chef/stable/ubuntu/precise main'
      key_url: 'https://packagecloud.io/gpg.key'
```
{: data-file=".travis.yml"}

#### Add APT Packages

List APT packages under the `addons.apt.packages` key:

```yaml
addons:
  apt:
    packages:
    - cmake
    - time
```
{: data-file=".travis.yml"}

> Note: When using APT sources and packages together, you need to make
> sure they are under the same key space in the YAML file. e.g.

```yaml
addons:
  apt:
    sources:
    - ubuntu-toolchain-r-test
    packages:
    - gcc-4.8
    - g++-4.8
```
{: data-file=".travis.yml"}

> Note: If `apt-get install` fails, the build is marked an error.

> You can also have a look at the [Apt](https://config.travis-ci.com/ref/job/addons/apt) section in our [Travis CI Build Config Reference](https://config.travis-ci.com/).

### Install Snap Packages with the Snaps Addon

You can install [snap](http://snapcraft.io/) packages using our Xenial or
Bionic images:

```yaml
dist: xenial
```
{: data-file=".travis.yml"}

or 

```yaml
dist: bionic
```
{: data-file=".travis.yml"}

The Ubuntu Snap store offers many packages directly maintained by upstream
developers, often with newer versions than the ones available in the Apt archive.

You can specify snaps as an array of snap specifications, each of which is
of the two possible forms:

1. The name of the snap, which will be passed on to the `snap install` without
  additional flags. For example,

      ```yaml
      dist: xenial
      addons:
        snaps:
          - hugo
      ```
      {: data-file=".travis.yml"}

    This results in:

      ```
      $ sudo snap install hugo
      ```

1. The map specifies how the snap should be installed. Possible keys are:
   `name`, `confinement`, and `channel`.
   The `confinement` key will be used to add `--classic` or `--devmode` flag,
   and `channel` will be passed to `--channel` flag.
   For example,

      ```yaml
      dist: xenial
      addons:
        snaps:
          - name: aws-cli
            confinement: classic # or devmode
            channel: latest/edge # will be passed to --channel flag
      ```
      {: data-file=".travis.yml"}

    This results in:

      ```
      $ sudo snap install aws-cli --classic --channel=latest/edge
      ```

    `confinement` and `channel` are optional.


## Install Packages on FreeBSD

To install packages that are not included in the default FreeBSD environment use `pkg` in the `before_install` step of your `.travis.yml`:

```yaml
before_install:
  - su -m root -c 'pkg install -y curl'
```
{: data-file=".travis.yml"}

For convenience, you can use the `pkg` addon in your `.travis.yml`. For example, to install go and curl:

```yaml
addons: 
 pkg: 
  - go 
  - curl
```
{: data-file=".travis.yml"}

## Install Dependencies on Multiple Operating Systems

If you're testing on both Linux and macOS, you can use both the APT addon and the Homebrew addon together. Each addon will only run on the appropriate platform:

```yaml
addons:
  apt:
    packages: foo
  homebrew:
    packages: bar
```
{: data-file=".travis.yml"}

If you're installing packages manually, use the `$TRAVIS_OS_NAME` variable to install dependencies separately for each OS:

```yaml
install:
  - if [ $TRAVIS_OS_NAME = linux ]; then sudo apt-get install foo; else brew install bar; fi
```
{: data-file=".travis.yml"}

## Install Projects from Source

Some dependencies can only be installed from a source package. The build may require a more recent version or a tool or library that's not available as a Ubuntu package.

You can easily include the build steps in either your .travis.yml or, and this is the recommended way, by running a script to handle the installation process.

Here's a simple example that installs CasperJS from a binary package:

```yaml
before_script:
  - wget https://github.com/n1k0/casperjs/archive/1.0.2.tar.gz -O /tmp/casper.tar.gz
  - tar -xvf /tmp/casper.tar.gz
  - export PATH=$PATH:$PWD/casperjs-1.0.2/bin/
```
{: data-file=".travis.yml"}

> Note that when you're updating the `$PATH` environment variable, that part can't be moved into a shell script, as it will only update the variable for the sub-process that's running the script.

To install something from source, you can follow similar steps. Here's an example to download, compile and install the protobufs library.

```yaml
install:
  - wget https://protobuf.googlecode.com/files/protobuf-2.4.1.tar.gz
  - tar -xzvf protobuf-2.4.1.tar.gz
  - pushd protobuf-2.4.1 && ./configure --prefix=/usr && make && sudo make install && popd
```
{: data-file=".travis.yml"}

These three commands can be extracted into a shell script, let's name it `install-protobuf.sh`:

```bash
#!/bin/sh
set -ex
wget https://protobuf.googlecode.com/files/protobuf-2.4.1.tar.gz
tar -xzvf protobuf-2.4.1.tar.gz
cd protobuf-2.4.1 && ./configure --prefix=/usr && make && sudo make install
```
{: data-file="install-protobuf.sh"}

Once it's added to the repository, you can run it from your .travis.yml:

```yaml
before_install:
  - ./install-protobuf.sh
```
{: data-file=".travis.yml"}

Note that the first version uses `pushd` and `popd` to ensure that after the `install` section completes, the working directory is returned to its original value.  This is not necessary in the shell script, as it runs in a sub-shell and so does not alter the original working directory.

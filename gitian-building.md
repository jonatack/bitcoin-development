# Gitian Building

Last updated: November 19, 2020

*This is based on fanquake's
[gitian-building](https://github.com/fanquake/core-review/blob/master/gitian-building/README.md)
resource that I initially updated in January 2020 with info and changes that I
found helpful when using it for the first time. Since then, I continue to update
this resource regularly.*

Bitcoin Core releases are [reproducibly built](https://reproducible-builds.org)
using [Gitian Builder](https://github.com/devrandom/gitian-builder).


If you have already set up your environment to build gitian signatures, you can
skip directly to [Make base VM for
gitian-building](#make-base-vm-for-gitian-building) below.


## Initial Dependencies

This is a simplified version of the dependencies documentation in
https://github.com/devrandom/gitian-builder#prerequisites -- don't hesitate to
refer to it for further info or if you get stuck.

### Linux

If `coreutils` is already included in your distribution, remove it from this
line:

```bash
sudo apt install apt-cacher-ng coreutils
```

### macOS

```bash
brew install coreutils
```

Gitian-builder needs `sha256sum`, which doesn't exist on macOS.

```bash
sudo ln -s /usr/local/bin/gsha256sum /usr/local/bin/sha256sum
```

```bash
brew cask install docker
```

### Docker info for all platforms

- To install Docker, see the information for your platform here: [Docker Engine
  overview](https://docs.docker.com/install/).
- For the [build script](#build-unsigned-sigs) below to function, Linux users
  may need to [allow Docker to run as a non-root
  user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user).
- Once Docker is installed, run it at least once (e.g. with `docker run
  hello-world` or by opening the app) so Docker can request and set up the
  necessary permissions.
- You might want to give Docker access to more CPUs, RAM, etc. This can be done
  by going to `Preferences -> Advanced` in the app and allocating as required.


## Initial Setup

```bash
mkdir gitian-building
pushd gitian-building
```

Fork and clone a copy of the
[gitian.sigs](https://github.com/bitcoin-core/gitian.sigs) repository.

```bash
# After forking https://github.com/bitcoin-core/gitian.sigs on GitHub
git clone https://github.com/your_github_username/gitian.sigs.git
```

Clone the other required repositories.

```bash
git clone https://github.com/bitcoin/bitcoin.git
git clone https://github.com/devrandom/gitian-builder.git
git clone https://github.com/bitcoin-core/bitcoin-detached-sigs.git # sigs for macOS and Windows only
```

Download the gitian inputs/dependencies into gitian-builder.
```bash
pushd gitian-builder
make -C ../bitcoin/depends download SOURCES_PATH=`pwd`/cache/common
mkdir -p inputs
wget -P inputs https://bitcoincore.org/cfields/osslsigncode-Backports-to-1.7.1.patch
wget -O osslsigncode-2.0.tar.gz -P inputs https://github.com/mtrojnar/osslsigncode/archive/2.0.tar.gz
wget -P inputs https://downloads.sourceforge.net/project/osslsigncode/osslsigncode/osslsigncode-1.7.1.tar.gz
wget -P inputs https://bitcoincore.org/depends-sources/sdks/MacOSX10.14.sdk.tar.gz
popd
```

## Make base VM for gitian-building

```bash
pushd gitian-builder
git pull # pull any updates
bin/make-base-vm --suite bionic --arch amd64 --docker
popd
```

## Set up and check out the branches to build

You can run the following `git tag` command from within your local bitcoin
repository to see the Bitcoin Core release tags and descriptions, ordered by
most recent last.

```bash
pushd bitcoin
git tag -n | sort -V
popd
```

Update the version and signer values with the version to build and your username.

```bash
export VERSION=0.21.0rc1
export SIGNER=your_username
export USE_DOCKER=1

pushd gitian.sigs
git checkout -b $SIGNER-$VERSION
popd

pushd bitcoin
git checkout v$VERSION
popd
```

## Build Unsigned Sigs

All of the instructions in this section are to be run from within the
`gitian-builder` directory:

```
pushd gitian-builder
```

#### Intro

The first time `depends` is built for a new version, it can take a *long* time,
as
[dependencies](https://github.com/bitcoin/bitcoin/tree/master/depends/packages)
are being built for all architectures and operating systems. Subsequent builds
will be much faster, as only Bitcoin Core is being compiled.

If you encounter any errors, it may be helpful to restart the base VM, or more
rarely, delete the cache in `gitian-builder/cache` and rebuild the depends:

```bash
# Don't do these unless you need to, because it takes much more time.
rm -r cache
make -C ../bitcoin/depends download SOURCES_PATH=`pwd`/cache/common
```

Follow build progress from the `gitian-builder` directory root using:

```bash
tail -f var/install.log # Setup
tail -f var/build.log # Building dependencies and Bitcoin Core
```

### Now, on to building the unsigned sigs

You would normally run all three of these (Linux, Windows, and macOS) and submit
the results in one PR.

Adjust `num-make` (number of cores, like for `make -j`) and `memory` (RAM) as
needed below. For instance, I have four cores and add one for the `num_make`
value.

The first build will be slow. The subsequent ones will go faster.

#### Linux

```bash
bin/gbuild --num-make 5 --memory 12000 --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-linux.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-linux --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-linux.yml
mv build/out/bitcoin-*.tar.gz build/out/src/bitcoin-*.tar.gz ../
```

#### Windows

```bash
bin/gbuild --num-make 5 --memory 12000 --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-win.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-win-unsigned --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-win.yml
mv build/out/bitcoin-*-win-unsigned.tar.gz inputs/bitcoin-win-unsigned.tar.gz
mv build/out/bitcoin-*.zip build/out/bitcoin-*.exe ../
```

#### macOS

```bash
bin/gbuild --num-make 5 --memory 12000 --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-osx.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-unsigned --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-osx.yml
mv build/out/bitcoin-*-osx-unsigned.tar.gz inputs/bitcoin-osx-unsigned.tar.gz
mv build/out/bitcoin-*.tar.gz build/out/bitcoin-*.dmg ../
```

```
popd
```

## Commit Unsigned Sigs

```bash
pushd gitian.sigs
git add .
git commit -m "$SIGNER $VERSION unsigned"
git push
popd
```

Go to https://github.com/bitcoin-core/gitian.sigs and open a PR from that
commit. Done 🍻


## Build Signed Sigs

Signed signatures can be built once the `detached sigs` are available in the
[detached-sigs
repo](https://github.com/bitcoin-core/bitcoin-detached-sigs/). They are often
pushed a day or so after the release is tagged and announced on the
#bitcoin-core-dev IRC channel with a message like "0.21.0 detached signatures
are up and tagged."

You would normally run both of these (macOS and Windows) and submit the results
in one PR. These build far faster than the builds for the unsigned signatures --
within a minute or so, or even a few seconds.

```
pushd gitian-builder
```

#### macOS

```bash
bin/gbuild -i --commit signature=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-signed --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-osx-signed ../bitcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
mv build/out/bitcoin-osx-signed.dmg ../bitcoin-${VERSION}-osx.dmg
```

#### Windows

```bash
bin/gbuild -i --commit signature=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-win-signer.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-win-signed --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-win-signer.yml
bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-win-signed ../bitcoin/contrib/gitian-descriptors/gitian-win-signer.yml
mv build/out/bitcoin-*win64-setup.exe ../bitcoin-${VERSION}-win64-setup.exe
```

```
popd
```

## Commit Signed Sigs

```bash
pushd gitian.sigs
git add .
git commit -m "$SIGNER $VERSION signed"
git push
popd
```

Go to https://github.com/bitcoin-core/gitian.sigs and open a PR from that
commit. Done 🍻


## One-off builds

You can invoke one-off builds using:

```bash
pushd gitian-builder

env USE_DOCKER=1 ./bin/gbuild -j8 -m 6000 \
--commit bitcoin=79218eae27f62e246d52dc6cda4e8846293e1c8e \
--url bitcoin=https://github.com/fanquake/bitcoin.git \
path/to/bitcoin/contrib/gitian-descriptors/gitian-osx.yml
```

Useful for testing PRs like [#17787](https://github.com/bitcoin/bitcoin/pull/17787).

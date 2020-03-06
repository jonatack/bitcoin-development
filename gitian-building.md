## Gitian Building

Last updated: March 6, 2020

*This is my updated version of fanquake's
[gitian-building](https://github.com/fanquake/core-review/blob/master/gitian-building/README.md) resource.*

Bitcoin Core releases are [reproducibly built](https://reproducible-builds.org)
using [Gitian Builder](https://github.com/devrandom/gitian-builder).


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

### Linux

```bash
sudo apt install coreutils # if coreutils is not already included in your distribution
```

### All platforms

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


## Setup

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
git clone https://github.com/devrandom/gitian-builder.git
git clone https://github.com/bitcoin/bitcoin.git
git clone https://github.com/bitcoin-core/bitcoin-detached-sigs.git # macOS and Windows only
```

Create the base VMs used for gitian-building.

```bash
pushd gitian-builder
bin/make-base-vm --suite bionic --arch amd64 --docker // For building 0.17 onwards
bin/make-base-vm --suite trusty --arch amd64 --docker // For building 0.15 & 0.16
popd
```

## Fetch Gitian Inputs

```bash
export VERSION=0.19.1
export SIGNER=your_username
export USE_DOCKER=1

pushd gitian.sigs
git checkout -b $SIGNER-$VERSION
popd

pushd bitcoin
git checkout v$VERSION
popd

pushd gitian-builder
make -C ../bitcoin/depends download SOURCES_PATH=`pwd`/cache/common
mkdir -p inputs
wget -P inputs https://bitcoincore.org/cfields/osslsigncode-Backports-to-1.7.1.patch
wget -P inputs https://downloads.sourceforge.net/project/osslsigncode/osslsigncode/osslsigncode-1.7.1.tar.gz
wget -P inputs https://bitcoincore.org/depends-sources/sdks/MacOSX10.14.sdk.tar.gz
```

If you want to gitian build for macOS, you'll need the macOS 10.11 SDK. You can
read the documentation
[here](https://github.com/bitcoin/bitcoin/blob/master/doc/build-osx.md#deterministic-macos-dmg-notes)
about how to create it. Once you have `MacOSX10.11.sdk.tar.gz`, place it in
`gitian-builder/inputs/`.


## Build Unsigned Sigs

You should still be in the `gitian-builder` repository. If not, run `pushd
gitian-builder`.

The first time `depends` is built for a new version, it can take a *long* time,
as [dependencies](https://github.com/bitcoin/bitcoin/tree/master/depends/packages)
are being built for all architectures and operating systems. Subsequent builds
will be much faster, as only Bitcoin Core is being compiled.

Follow build progress using:

```bash
tail -f var/install.log # Setup
tail -f var/build.log # Building dependencies and Bitcoin Core
```

Adjust `num-make` and `memory` as needed below.

### Linux

```bash
bin/gbuild --num-make 5 --memory 12000 --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-linux.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-linux --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-linux.yml
mv build/out/bitcoin-*.tar.gz build/out/src/bitcoin-*.tar.gz ../
```

### macOS

```bash
bin/gbuild --num-make 5 --memory 12000 --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-osx.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-unsigned --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-osx.yml
mv build/out/bitcoin-*-osx-unsigned.tar.gz inputs/bitcoin-osx-unsigned.tar.gz
mv build/out/bitcoin-*.tar.gz build/out/bitcoin-*.dmg ../
```

### Windows

```bash
bin/gbuild --num-make 5 --memory 12000 --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-win.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-win-unsigned --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-win.yml
mv build/out/bitcoin-*-win-unsigned.tar.gz inputs/bitcoin-win-unsigned.tar.gz
mv build/out/bitcoin-*.zip build/out/bitcoin-*.exe ../
```

```
popd
```

## Commit Unsigned Sigs

```bash
pushd gitian.sigs
git add .
git commit -m "$SIGNER $VERSION unsigned sigs"
git push
popd
```

## Build Signed Sigs - macOS and Windows only

Signed signatures can be built once the `detached sigs` are available in the
[detached-sigs repo](https://github.com/bitcoin-core/bitcoin-detached-sigs/).

```
pushd gitian-builder
```

### macOS

```bash
bin/gbuild -i --commit signature=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-signed --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
bin/gverify -v -d ../gitian.sigs/ -r ${VERSION}-osx-signed ../bitcoin/contrib/gitian-descriptors/gitian-osx-signer.yml
mv build/out/bitcoin-osx-signed.dmg ../bitcoin-${VERSION}-osx.dmg
```

### Windows

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
git commit -m "$SIGNER $VERSION signed sigs"
git push
popd
```

Done 🍻


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

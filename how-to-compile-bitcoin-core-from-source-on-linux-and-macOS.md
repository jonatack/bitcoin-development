How to compile Bitcoin Core from source on Linux Debian and macOS and run the unit and functional tests
-------------------------------------------------------------------------------------------------------
Last updated: 6 January 2020

*Hi! This article now lives at
[https://jonatack.github.io/articles/how-to-compile-bitcoin-core-and-run-the-tests](https://jonatack.github.io/articles/how-to-review-pull-requests-in-bitcoin-core).
I will continue to update it there. Cheers.*

This is a simplified summary of the various docs in
https://github.com/bitcoin/bitcoin/tree/master/doc. Don't hesitate to read them
for more information.

All steps are to be run from your terminal emulator, i.e. the command line.

1. Ensure the dependencies are installed:

    - Linux: `sudo apt-get install build-essential libtool autotools-dev
      automake pkg-config bsdmainutils python3 libssl-dev libevent-dev
      libboost-system-dev libboost-filesystem-dev libboost-chrono-dev
      libboost-test-dev libboost-thread-dev libminiupnpc-dev libzmq3-dev
      libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools
      libprotobuf-dev protobuf-compiler git ccache libqrencode-dev`

    - macOS (with command line tools and Homebrew already installed): `brew
      install automake berkeley-db4 libtool boost miniupnpc openssl pkg-config
      protobuf python qt libevent qrencode ccache`

2. Download the Bitcoin source files by git cloning the repository:
    - `git clone https://github.com/bitcoin/bitcoin.git`

3. Install Berkeley DB (BDB) v4.8, a backward-compatible version needed for the
   wallet, using the script in /contrib:
    - Enter your local copy of the bitcoin repository: `cd bitcoin`
    - Now that you are in the root of the bitcoin repository, run
      ``./contrib/install_db4.sh `pwd` ``
    - Take note of the instructions displayed in the terminal at the end of the
      BDB installation process:
```
      db4 build complete.

      When compiling bitcoind, run `./configure` in the following way:

      export BDB_PREFIX='<PATH-TO>/db4'

      ./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include" ...
```

4. Compile from a tagged release branch instead of master, unless you want to
   test the bleeding edge:
    - `git tag -n | sort -V` to see tags and descriptions ordered by most recent last
    - `git checkout <TAG>` to use a tagged release, for example: `git checkout v0.19.0.1`

5. Compile Bitcoin from source:
    - `./autogen.sh`
    - `export BDB_PREFIX='<PATH-TO>/db4'` (you can use the output from the BDB
      build above)
    - `./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8"
      BDB_CFLAGS="-I${BDB_PREFIX}/include"`
    - `make`, or if you have multiple threads on your machine, you can tell
      `make` to use all of them and reduce compile time significantly with
    - `make -j"$(($(nproc)+1))"` on Linux, or
    - `make -j"$(($(sysctl -n hw.physicalcpu)+1))"` on macOS

    You can run `./configure --help` to see the various configuration options.

    If you build frequently from source (e.g. for testing pull requests), as
    long as you don't need to change the configuration options you can skip
    directly to the `make` step for subsequent builds.

    Be sure to use `ccache` to speed up your builds. You can also gain time by
    building only what you need. See the Bitcoin Core [productivity
    notes](https://github.com/bitcoin/bitcoin/blob/master/doc/productivity.md)
    for more.

6. Run the unit tests:
    - `make check`, or
    - `make -j"$(($(nproc)+1))" check` to use multithreading on Linux, or
    - `make -j"$(($(sysctl -n hw.physicalcpu)+1))" check` on macOS

7. Run the functional tests. From the repository root:
    - `test/functional/test_runner.py` to run the standard test suite
    - `test/functional/test_runner.py --extended` to run the extended test suite
    - `test/functional/test_runner.py --help` to see the various options for running tests

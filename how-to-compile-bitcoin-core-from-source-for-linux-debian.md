How to compile Bitcoin Core from source on Linux Debian and run the unit and functional tests
---------------------------------------------------------------------------------------------
Last updated: 16 March 2019

This is a simplified compilation of the various docs in https://github.com/bitcoin/bitcoin/tree/master/doc. Don't hesitate to read them for more information.

All steps are to be run from your terminal emulator, i.e. the command line.

1. Ensure the dependencies are installed:
    - `sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3 libssl-dev libevent-dev libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-test-dev libboost-thread-dev libminiupnpc-dev libzmq3-dev libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler git ccache`

2. Download the Bitcoin source files by git cloning the repository:
    - `git clone https://github.com/bitcoin/bitcoin.git`

3. Install Berkeley DB (BDB) v4.8, a backward-compatible version needed for the wallet, using the script in /contrib:
    - Enter your local copy of the bitcoin repository: `cd bitcoin`
    - Now that you are in the root of the bitcoin repository, run ``./contrib/install_db4.sh `pwd` ``
    - Take note of the instructions displayed in the terminal at the end of the BDB installation process:
```
      db4 build complete.

      When compiling bitcoind, run `./configure` in the following way:

      export BDB_PREFIX='<PATH-TO>/db4'

      ./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include" ...
```

4. [Recommended] Compile from a tagged release branch instead of master, unless you really want to test the bleeding edge:
    - `git tag -n | sort -V` to see tags and descriptions ordered by most recent last
    - `git checkout <TAG>` to use a tagged release, for example: `git checkout v0.18.0rc2`

5. Compile Bitcoin from source, optionally with lcov and gprof enabled:
    - `./autogen.sh`
    - Use the output from the BDB build above for the exact export command: `export BDB_PREFIX='<PATH-TO>/db4'`
    - `./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include" --enable-lcov --enable-gprof`
    - `make`, or if you have multiple threads on your machine, you can tell `make` to utilize all of them and reduce compile time significantly with: `make -j"$(($(nproc)+1))"`

6. Run unit tests:
    - Unit tests only: `make check`
    - Unit tests with coverage report (if lcov was enabled): `make cov`

7. Run functional tests:
    - `test/functional/test_runner.py` or `test/functional/test_runner.py --extended`

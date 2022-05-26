# Scripted Diffs

Description from `/doc/developer-notes.md` in the repo:

For reformatting and refactoring commits where the changes can be easily
automated using a bash script, we use scripted-diff commits. The bash script is
included in the commit message and our CI checks that the result of the script
is identical to the commit. This aids reviewers since they can verify that the
script does exactly what it's supposed to do. It is also helpful for rebasing
(since the same script can just be re-run on the new master commit).

To create a scripted-diff:

- start the commit message with `scripted-diff:` (and then a description of the
  diff on the same line)

- in the commit message include the bash script between lines containing just
  the following text:

- `-BEGIN VERIFY SCRIPT-`

- `-END VERIFY SCRIPT-`

- for the first line of the bash script specifying the interpreter (the shebang)
  use `#!/usr/bin/env bash` instead of the obsolete `#!/bin/bash`.

-----

The scripted-diff is verified by the script check tool:
```
test/lint/commit-script-check.sh
```
The tool's default behavior when supplied with a commit is to verify all
scripted-diffs from the beginning of time up to said commit. Internally, the
tool passes the first supplied argument to `git rev-list --reverse` to determine
which commits to verify script-diffs for, ignoring commits that don't conform to
the commit message format described above.

For development, it might be more convenient to verify all scripted-diffs in a
range `A..B`, for example:
```
test/lint/commit-script-check.sh origin/master..HEAD
```

## Examples

scripted-diff: remove duplicate categories from logging output
```
-BEGIN VERIFY SCRIPT-
s() { git grep -l "$1" src | xargs sed -i "s/$1/$2/g"; }
s 'BCLog::TOR, "tor: '       'BCLog::TOR, "'
s 'BCLog::I2P, "I2P: '       'BCLog::I2P, "'
s 'BCLog::NET, "net: '       'BCLog::NET, "'
s 'BCLog::ZMQ, "zmq: '       'BCLog::ZMQ, "'
s 'BCLog::PRUNE, "Prune: '   'BCLog::PRUNE, "'
-END VERIFY SCRIPT-
```
-----

scripted-diff: More accurate names to avoid confusion
```
-BEGIN VERIFY SCRIPT-
sed -i -e 's/UsedDestination/SpentKey/g' $(git grep -l 'UsedDestination' ./src)
-END VERIFY SCRIPT-
```
-----

scripted-diff: Sort test includes
```
-BEGIN VERIFY SCRIPT-
# Mark all lines with #includes
sed -i --regexp-extended -e 's/(#include <.*>)/\1 /g' $(git grep -l '#include' ./src/bench/ ./src/test ./src/wallet/test/)
# Sort all marked lines
git diff -U0 | ./contrib/devtools/clang-format-diff.py -p1 -i -v
-END VERIFY SCRIPT-
```
-----

scripted-diff: Bump copyright headers
```
-BEGIN VERIFY SCRIPT-
./contrib/devtools/copyright_header.py update ./
-END VERIFY SCRIPT-
```
-----

scripted-diff: Replace fprintf with tfm::format

sed -i --regexp-extended -e 's/fprintf\(std(err|out), /tfm::format(std::c\1, /g' $(git grep -l 'fprintf(' -- ':(exclude)src/crypto' ':(exclude)src/leveldb' ':(exclude)src/univalue' ':(exclude)src/secp256k1')

-----

scripted-diff: stop using the gArgs wrappers

find src/ -name "*.cpp" ! -wholename "src/util.h" ! -wholename "src/util.cpp" | xargs perl -i -pe 's/(?<!\.)(ParseParameters|ReadConfigFile|IsArgSet|(Soft|Force)?(Get|Set)(|Bool|)Arg(s)?)\(/gArgs.\1(/g'

-----

scripted-diff: gitian: Use REFERENCE_DATETIME directly

    sed -i 's#\$REFERENCE_DATE\\\\\\ \$REFERENCE_TIME#\$REFERENCE_DATETIME#g' contrib/gitian-descriptors/*

-----

scripted-diff: Complete the move from CCriticalSection to identical RecursiveMutex

    git grep -l "CCriticalSection" ":(exclude)src/sync.h" | xargs sed -i "s/CCriticalSection/RecursiveMutex/g"

-----

https://github.com/bitcoin/bitcoin/pull/19373

    sed -i --regexp-extended -e 's/HexStr\(([^(]+)\.begin\(\), *([^(]+)\.end\(\)\)/HexStr(\1)/g' $(git grep -l HexStr)

-----

https://github.com/bitcoin/bitcoin/pull/19953/commits/d17bebdf9b42b6f823aced1d3768879707d94847

scripted-diff: put ECDSA in name of signature functions

In preparation for adding Schnorr versions of `CheckSig`, `VerifySignature`, and
`ComputeEntry`, give them an ECDSA specific name.

-BEGIN VERIFY SCRIPT-
sed -i 's/CheckSig(/CheckECDSASignature(/g' $(git grep -l CheckSig ./src)
sed -i 's/VerifySignature(/VerifyECDSASignature(/g' $(git grep -l VerifySignature ./src)
sed -i 's/ComputeEntry(/ComputeEntryECDSA(/g' $(git grep -l ComputeEntry ./src)
-END VERIFY SCRIPT-

-----

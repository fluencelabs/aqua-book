# Changelog

Aqua compiler's versioning scheme is the following: `0.BREAKING.ENHANCING.RELEASE`

* `0` shows that Aqua does not meet its vision yet, so syntax and semantics can change fastly
* `BREAKING` part is incremented for each breaking change when old `.aqua` files need to be updated to compile with the new version
* `ENHANCING` part is incremented for every syntax addition
* `RELEASE` is the release number, shows internal compiler changes, bugfixes that keep the language untouched

### [0.2.1](https://github.com/fluencelabs/aqua/releases/tag/0.2.1) – August 31, 2021

* Javascript build of the compiler is now distributed via NPM: to run without Java, use `aqua-js` command \([\#256](https://github.com/fluencelabs/aqua/pull/256)\)
* Now dots are allowed in the module declarations: `module Space.Module` & many bugfixes \([\#258](https://github.com/fluencelabs/aqua/pull/258)\)

### [0.2.0](https://github.com/fluencelabs/aqua/releases/tag/0.2.0) – August 27, 2021

* Now the compiler emits AIR with the new `(ap` instruction, hence it's not backwards compatible \([\#241](https://github.com/fluencelabs/aqua/pull/241)\)
* Many performance optimizations and bugfixes \([\#255](https://github.com/fluencelabs/aqua/pull/255), [\#254](https://github.com/fluencelabs/aqua/pull/254), [\#252](https://github.com/fluencelabs/aqua/pull/252), [\#249](https://github.com/fluencelabs/aqua/pull/249)\)

### [0.1.14](https://github.com/fluencelabs/aqua/releases/tag/0.1.14) – August 20, 2021

* Aqua file header changes: `module`, `declares`, `use`, `export` expressions \([\#245](https://github.com/fluencelabs/aqua/pull/245)\), see [Imports and Exports](language/header.md) for the docs. 
* Experimental Scala.js build of the compiler \([\#247](https://github.com/fluencelabs/aqua/pull/247)\)

### [0.1.13](https://github.com/fluencelabs/aqua/releases/tag/0.1.13) – August 10, 2021

* Functions can export \(return\) several values, see [\#229](https://github.com/fluencelabs/aqua/pull/229)
* Internal changes: migrate to Scala3 \([\#228](https://github.com/fluencelabs/aqua/pull/228)\), added Product type \([\#168](https://github.com/fluencelabs/aqua/pull/225)\)

### [0.1.12](https://github.com/fluencelabs/aqua/releases/tag/0.1.12) – August 4, 2021

* Can have functions consisting of a return operand only, returning a literal or an argument

### [0.1.11](https://github.com/fluencelabs/aqua/releases/tag/0.1.11) – August 3, 2021

* Added `host_peer_id` , a predefined constant that points on the relay if Aqua compilation is configured so, and on `%init_peer_id%` otherwise, see [\#218](https://github.com/fluencelabs/aqua/issues/218).

### 0.1.10 – July 26, 2021

* Added `<<-` operator to push a value into a stream, see \#[214](https://github.com/fluencelabs/aqua/pull/214), [\#209](https://github.com/fluencelabs/aqua/issues/209).




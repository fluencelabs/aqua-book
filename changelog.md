# Changelog

Aqua compiler's versioning scheme is the following: `0.BREAKING.ENHANCING.RELEASE`

* `0` shows that Aqua does not meet its vision yet, so syntax and semantics can change quickly
* `BREAKING` part is incremented for each breaking change when old `.aqua` files need to be updated to compile with the new version
* `ENHANCING` part is incremented for every syntax addition
* `RELEASE` is the release number, shows internal compiler changes, bugfixes that keep the language untouched

### [0.5.0](https://github.com/fluencelabs/aqua/releases/tag/0.5.0) – November 24, 2021

* Breaking semantic change: [Stream restrictions](language/crdt-streams.md#stream-restrictions). This fixes many obscure bugs which happened when using streams inside `for` cycles ([#321](https://github.com/fluencelabs/aqua/issues/321))
* This version of Aqua is not compatible with `fldist` so far (cannot run the emitted `AIR` via `fldist`). Use `aqua --run` to run Aqua instead ([#358](https://github.com/fluencelabs/aqua/pull/358))
* Added timeout parameter support for `aqua --run` ([#360](https://github.com/fluencelabs/aqua/pull/360))
* You need to update [FluenceJS to 0.15.0](changelog.md#0.5.0-november-24-2021)+ and [Fluence Node to v0.0.23](https://github.com/fluencelabs/node-distro/releases/tag/v0.0.23)+ for Aqua 0.5 support, previous versions will not work.

### [0.4.1](https://github.com/fluencelabs/aqua/releases/tag/0.4.1) – November 10, 2021

* New language feature: [closures](language/closures.md) ([#327](https://github.com/fluencelabs/aqua/pull/327))
* New CLI option `--scheduled` to compile Aqua for the Fluence's [Script Storage](libraries/aqua-lib.md) ([#355](https://github.com/fluencelabs/aqua/pull/355))
* Bugfixes for using streams to construct more complex streams ([#277](https://github.com/fluencelabs/aqua/issues/277))
* Better errors rendering ([#322](https://github.com/fluencelabs/aqua/issues/322), [#337](https://github.com/fluencelabs/aqua/issues/337))
* Bugfix for comparing Option types ([#343](https://github.com/fluencelabs/aqua/issues/343))

### [0.4.0](https://github.com/fluencelabs/aqua/releases/tag/0.4.0) – October 25, 2021

* Now Aqua compiler emits JS/TS code for [Fluence JS 0.14](https://www.npmjs.com/package/@fluencelabs/fluence). The new JS/TS SDK is heavily rewritten to [support async service functions declaration](https://doc.fluence.dev/docs/fluence-js/3\_in\_depth#using-asynchronous-code-in-callbacks). It also embeds a deeply refactored [AquaVM](https://github.com/fluencelabs/aquavm). ([#334](https://github.com/fluencelabs/aqua/pull/334))
* Various bugfixes for AIR generation and the compiler behavior ([#328](https://github.com/fluencelabs/aqua/pull/328), [#335](https://github.com/fluencelabs/aqua/pull/335), [#336](https://github.com/fluencelabs/aqua/pull/336), [#338](https://github.com/fluencelabs/aqua/pull/338))

### [0.3.2](https://github.com/fluencelabs/aqua/releases/tag/0.3.2) – October 13, 2021

* Experimental feature: now can run Aqua from Aqua CLI ([#324](https://github.com/fluencelabs/aqua/pull/324)):

```
aqua run -i aqua/caller.aqua -f "callFunc(\"arg1\",\"arg2\")"
```

* Many performance-related updates, compiler now runs faster ([#308](changelog.md#0.3.2-october-13-2021), [#324](https://github.com/fluencelabs/aqua/pull/324))&#x20;
* UX improvements for CLI and JS/TS backend ([#307](https://github.com/fluencelabs/aqua/pull/307), [#313](https://github.com/fluencelabs/aqua/pull/313), [#303](https://github.com/fluencelabs/aqua/pull/303), [#305](https://github.com/fluencelabs/aqua/pull/305), [#301](https://github.com/fluencelabs/aqua/pull/301), [#302](https://github.com/fluencelabs/aqua/pull/302))

### [0.3.1](https://github.com/fluencelabs/aqua/releases/tag/0.3.1) – September 13, 2021

* Now `.aqua` extension in imports is optional: you may `import "file.aqua"` or just `import "file"` with the same effect ([#292](https://github.com/fluencelabs/aqua/pull/292))
* CLI improvements: `--dry` run ([#290](https://github.com/fluencelabs/aqua/pull/290)), output directory is created if not present ([#287](https://github.com/fluencelabs/aqua/pull/287))
* Many bugfixes: for imports ([#289](https://github.com/fluencelabs/aqua/pull/289)), TypeScript backend ([#285](https://github.com/fluencelabs/aqua/pull/285), [#294](https://github.com/fluencelabs/aqua/pull/294), [#298](https://github.com/fluencelabs/aqua/pull/298)), and language semantics ([#275](https://github.com/fluencelabs/aqua/issues/275)).

### [0.3.0](https://github.com/fluencelabs/aqua/releases/tag/0.3.0) – September 8, 2021

* TypeScript output of the compiler now targets a completely rewritten [TypeScript SDK](https://doc.fluence.dev/docs/js-sdk) ([#251](https://github.com/fluencelabs/aqua/pull/251))
* Constants are now `UPPER_CASED`, including always-available `HOST_PEER_ID` and `INIT_PEER_ID` ([#260](https://github.com/fluencelabs/aqua/pull/260))
* The compiler is now distributed as [@fluencelabs/aqua](https://www.npmjs.com/package/@fluencelabs/aqua) package (was `aqua-cli`) ([#278](https://github.com/fluencelabs/aqua/pull/278))
* `aqua` is the name of the compiler CLI command now (was `aqua-cli`) ([#278](https://github.com/fluencelabs/aqua/pull/278))
* JVM version of the compiler is now available with `aqua-j` command; JS build is called by default – so no more need to have JVM installed ([#278](https://github.com/fluencelabs/aqua/pull/278))
* Now you can have a file that contains only a header with imports, uses, declares, and exports, and no new definitions ([#274](https://github.com/fluencelabs/aqua/pull/274))

### [0.2.1](https://github.com/fluencelabs/aqua/releases/tag/0.2.1) – August 31, 2021

* Javascript build of the compiler is now distributed via NPM: to run without Java, use `aqua-js` command ([#256](https://github.com/fluencelabs/aqua/pull/256))
* Now dots are allowed in the module declarations: `module Space.Module` & many bugfixes ([#258](https://github.com/fluencelabs/aqua/pull/258))

### [0.2.0](https://github.com/fluencelabs/aqua/releases/tag/0.2.0) – August 27, 2021

* Now the compiler emits AIR with the new `(ap` instruction, hence it's not backwards compatible ([#241](https://github.com/fluencelabs/aqua/pull/241))
* Many performance optimizations and bugfixes ([#255](https://github.com/fluencelabs/aqua/pull/255), [#254](https://github.com/fluencelabs/aqua/pull/254), [#252](https://github.com/fluencelabs/aqua/pull/252), [#249](https://github.com/fluencelabs/aqua/pull/249))

### [0.1.14](https://github.com/fluencelabs/aqua/releases/tag/0.1.14) – August 20, 2021

* Aqua file header changes: `module`, `declares`, `use`, `export` expressions ([#245](https://github.com/fluencelabs/aqua/pull/245)), see [Imports and Exports](language/header/) for the docs.&#x20;
* Experimental Scala.js build of the compiler ([#247](https://github.com/fluencelabs/aqua/pull/247))

### [0.1.13](https://github.com/fluencelabs/aqua/releases/tag/0.1.13) – August 10, 2021

* Functions can export (return) several values, see [#229](https://github.com/fluencelabs/aqua/pull/229)
* Internal changes: migrate to Scala3 ([#228](https://github.com/fluencelabs/aqua/pull/228)), added Product type ([#168](https://github.com/fluencelabs/aqua/pull/225))

### [0.1.12](https://github.com/fluencelabs/aqua/releases/tag/0.1.12) – August 4, 2021

* Can have functions consisting of a return operand only, returning a literal or an argument

### [0.1.11](https://github.com/fluencelabs/aqua/releases/tag/0.1.11) – August 3, 2021

* Added `host_peer_id` , a predefined constant that points on the relay if Aqua compilation is configured so, and on `%init_peer_id%` otherwise, see [#218](https://github.com/fluencelabs/aqua/issues/218).

### 0.1.10 – July 26, 2021

* Added `<<-` operator to push a value into a stream, see #[214](https://github.com/fluencelabs/aqua/pull/214), [#209](https://github.com/fluencelabs/aqua/issues/209).


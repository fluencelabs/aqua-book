# Changelog

Aqua compiler's versioning scheme is the following: `0.BREAKING.ENHANCING.RELEASE`

* `0` shows that Aqua does not meet its vision yet, so syntax and semantics can change fastly
* `BREAKING` part is incremented for each breaking change, when old .aqua files needs to be updated to compile with the new version
* `ENHANCING` part is incremented for every syntax addition
* `RELEASE` is the release number, shows internal compiler changes, bugfixes that keep the language untached

### 0.1.10 – July 26, 2021

* Added `<<-` operator to push a value into a stream, see \#[214](https://github.com/fluencelabs/aqua/pull/214), [\#209](https://github.com/fluencelabs/aqua/issues/209)




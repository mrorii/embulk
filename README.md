# What's Embulk?

Embulk is a plugin-based parallel bulk data loader that helps **data transfer** between various **storages**, **databases**, **NoSQL** and **cloud services**.

You can release plugins to share your efforts of data cleaning, error handling, transaction control, and retrying. Packaging effrots into plugins **brings OSS-style development to the data scripts** which **was tend to be one-time adhoc scripts**.

[Embulk, an open-source plugin-based parallel bulk data loader](http://www.slideshare.net/frsyuki/embuk-making-data-integration-works-relaxed) at Slideshare

[![Embulk](https://gist.githubusercontent.com/frsyuki/f322a77ee2766a508ba9/raw/e8539b6b4fda1b3357e8c79d3966aa8148dbdbd3/embulk-overview.png)](http://www.slideshare.net/frsyuki/embuk-making-data-integration-works-relaxed/12)

# Document

* [Quick Start](#quick-start)
  * [Using plugins](#using-plugins)
  * [Using plugin bundle](#using-plugin-bundle)
  * [Releasing plugins to RubyGems](#releasing-plugins-to-rubygems)
  * [Resuming a failed transaction](#resuming-a-failed-transaction)
* [Embulk Development](#embulk-development)
  * [Build](#build)
  * [Release](#release)

## Quick Start

The single-file package is the simplest way to try Embulk. You can download the latest embulk-VERSION.jar from [the releases page](https://bintray.com/embulk/maven/embulk/view#files) and run it with java.

### Linux & Mac & BSD

Embulk is a Java application. Please make sure that you installed [Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html).

Following 4 commands install embulk to your home directory:

```
curl --create-dirs -o ~/.embulk/bin/embulk -L https://bintray.com/artifact/download/embulk/maven/embulk-0.5.4.jar
chmod +x ~/.embulk/bin/embulk
echo 'export PATH="$HOME/.embulk/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Next step: [Trying examples](#trying-examples)

### Windows

Embulk is a Java application. Please make sure that you installed [Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html).

You can assume the jar file is a .bat file.

```
PowerShell -Command "& {Invoke-WebRequest https://bintray.com/artifact/download/embulk/maven/embulk-0.5.4.jar -OutFile embulk.bat}"
```

Next step: [Trying examples](#trying-examples)

### Trying examples

Let's load a CSV file, for example. `embulk example` subcommand generates a csv file and config file for you.

```
embulk example ./try1
embulk guess   ./try1/example.yml -o config.yml
embulk preview config.yml
embulk run     config.yml
```

Next step: [Using plugins](#using-plugins)

### Using plugins

You can use plugins to load data from/to various systems and file formats.
An example is [embulk-output-postgres-json](https://github.com/frsyuki/embulk-output-postgres-json) plugin. It outputs data into PostgreSQL server using "json" column type.

```
embulk gem install embulk-output-postgres-json
embulk gem list
```

You can find plugins at the [list of plugins by category](http://www.embulk.org/plugins/).

### Using plugin bundle

`embulk bundle` subcommand creates (or updates if already exists) a private (isolated) bundle of a plugins.
You can use the bundle using `-b <bundle_dir>` option. `embulk bundle` also generates some example plugins to \<bundle_dir>/embulk/\*.rb directory.

See the generated \<bundle_dir>/Gemfile file how to plugin bundles work.

```
embulk bundle ./embulk_bundle
embulk guess  -b ./embulk_bundle ...
embulk run    -b ./embulk_bundle ...
```

### Releasing plugins to RubyGems

TODO: documents

```
embulk-plugin-xyz
```

### Resuming a failed transaction

Embulk supports resuming failed transactions.
To enable resuming, you need to start transaction with `-r PATH` option:

```
embulk run config.yml -r resume-state.yml
```

If the transaction fails, embulk stores state some states to the yaml file. You can retry the transaction using exactly same command:

```
embulk run config.yml -r resume-state.yml
```

If you giveup to resume the transaction, you can use `embulk cleanup` subcommand to delete intermediate data:

```
embulk cleanup config.yml -r resume-state.yml
```


## Embulk Development

### Build

```
./gradlew cli  # creates pkg/embulk-VERSION.jar
./gradlew gem  # creates pkg/embulk-VERSION.gem
```

You can see JaCoCo's test coverage report at `${project}/build/reports/tests/index.html`
You can see Findbug's report at `${project}/build/reports/findbug/main.html`  # FIXME coverage information is not included somehow

You can use `classpath` task to use `./bin/embulk` for development:

```
./gradlew classpath  # -x test: skip test
./bin/embulk
```

To deploy artifacts to your local maven repository at ~/.m2/repository/:

```
./gradlew install
```

To compile the source code of embulk-core project only:

```
./gradlew :embulk-core:compileJava
```

Task `dependencies` shows dependency tree of embulk-core project:

```
./gradlew :embulk-core:dependencies
```

### Documents

Embulk uses Sphinx, YARD (Ruby API) and JavaDoc (Java API) for document generation.

```
brew install python
pip install sphinx
gem install yard
./gradlew site
# documents are: embulk-docs/build/html
```

### Release

You need to add your bintray account information to ~/.gradle/gradle.properties

```
bintray_user=(bintray user name)
bintray_api_key=(bintray api key)
```

Run following commands and follow its instruction:

```
./gradlew set_version -Pto=$VERSION
```

```
./gradlew releaseCheck
./gradlew release
git commit -am v$VERSION
git tag v$VERSION
```

See also:
* [Bintray](https://bintray.com)
* [How to acquire bintray API Keys](https://bintray.com/docs/usermanual/interacting/interacting_apikeys.html)


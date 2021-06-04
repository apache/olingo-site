Title: Release Documentation

# Apache Olingo Release Documentation

---

### Introduction

This document describes the release guidelines for Apache Olingo. It heavily refers
to [standard Apache procedures to release](http://maven.apache.org/developers/release/apache-release.html)
 Maven based projects at Apache.

### Build Environments

Apache Olingo is built and released with [Maven3](http://maven.apache.org) and uses
the <a href="http://svn.apache.org/repos/asf/maven/pom/tags/apache-13/pom.xml">Apache POM version 13</a>.

### Release Artifacts

An Apache Olingo release consists of:

  * All POMs/JARs/WARs built as part of the standard Maven build process. For
an overview on the released modules see artifacts with groupId
_org.apache.olingo_ in the [Apache Maven Repository](https://repository.apache.org/index.html#nexus-search;gav~org.apache.olingo~~~~).
In detail, per every module, where applicable, the following artifacts are produced:
   * **Main artifact**: `<artifactId>-<version>.<ext>`
   * **Source artifact**: `<artifactId>-<version>-sources.<ext>`
   * **Javadoc artifact**: `<artifactId>-<version>-javadoc.<ext>`
   * **POM**: `<artifactId>-<version>.pom`



  Also the following additional *distribution commodity packages* are
provided as part of the release:

   *  `org.apache.olingo-olingo-odata2-parent-${version}-source-release.${ext}` <br/> A source-release
bundle containing all files the sources necessary to build all other artifacts. <br/> **Package formats**: zip.

   *  `org.apache.olingo-olingo-odata2-dist-${version}-lib.${ext}` <br/> A bundle containing the OData2 core
library and dependencies required to implement an OData V2 processor. <br/> **Package formats**: zip.

   *  `org.apache.olingo-olingo-odata2-dist-${version}-javadoc.${ext}` <br/> A bundle
containing JavaDoc of the OData2 library API and annotations, the
JPA processor API as well as additional documentation and reference scenario
examples. <br/> **Package formats**: zip.

   *  `org.apache.olingo-olingo-odata2-dist-${version}-jpa.${ext}` <br/> A bundle containing the OData2 JPA
processor and dependencies required to implement an OData V2 processor. <br/> **Package formats**: zip.

   *  `org.apache.olingo-olingo-odata2-dist-${version}-janos.${ext}` <br/> A bundle containing the OData2 Java Annotation
processor and dependencies required to implement an OData V2 processor using Java Annotations. <br/> **Package formats**: zip.

   *  `org.apache.olingo-olingo-odata2-dist-${version}-ref.${ext}` <br/> A bundle containing ready-to-depoly WAR
files of the OData V2 reference scenarios implementations for the core library and the JPA processor. <br/> **Package formats**: zip.

### Documentation and JavaDoc

The documentation that will be part of the release must match the code.
All examples in the documentation must work. The Java package documentation must be
up-to-date. Release independend documentation is maintained on the [Apache Olingo Documentation][1] page.

### Preparation

##### Release Manager

A release manager must be appointed for a release. He or she is in charge of the release process,
following the guidelines and eventually generating the release artifacts.
The release manager might tailor the process for a specific release.

##### Version

The Olingo community decides if the release will be a major or a minor release and
agrees on a version number.

    mvn versions:set -DnewVersion=1.0.0-RC01
    find . -name '*.versionsBackup' -type f -delete
    git add .
    git commit -am 'Issue OLINGO-25 - make release - set version 1.0.0-RC01'
    git tag -f 1.0.0-RC01

    mvn versions:set -DnewVersion=1.1.0-SNAPSHOT
    find . -name '*.versionsBackup' -type f -delete
    git add .
    git commit -am 'Issue OLINGO-25 - make release - set version 1.1.0-SNAPSHOT'
    git push
    git push --tags


##### Open Issues

There must not be any open JIRA issues for this release. There might be open issues for
future releases. Check with: [fix for version view][2]

##### Unit Tests and Integration Tests

All unit tests and integration tests must succeed on a
clean machine (starting with an empty local Maven repository). The following Maven
execution will run all unit and integration tests:

    mvn clean install

##### Apache License and Code Style

Each source code file must have a current ASF license header. The source
code should follow the Apache Olingo code style. For verification run following
Maven execution

    mvn clean install -Pbuild.quality

##### Packaging

NOTICE, LICENSE and DISCLAIMER must be present in all bundles and must be up-to-date.

Remote resources are provided by the ASF and the Maven `remote-resources-plugin` is
configured in the parent pom of the project.

~~~xml
<resourceBundle>org.apache:apache-jar-resource-bundle:1.4</resourceBundle>
<resourceBundle>org.apache:apache-disclaimer-resource-bundle:1.1</resourceBundle>
~~~

The Maven module `odata2-dist` is responsible to package convenience distribution zip files
using the assembly plugin. The distributions are created with a release build `mvn clean install -Papache-release -Dgpg.passphrase="yourPassphraseHere"`

##### SHA for distribution packages

SHA files are created manually for distribution packages:

    gpg --print-md SHA512 ${filename}.zip > ${filename}.zip.sha512

DO NOT generate md5 files.

##### Release Tag

A tag has to be created for every release candidate. The naming rule
for the tags is olingo-${version}-RCxx. This is created as
part of the Maven release process. The tag will be renamed to the
final version number upon vote approval.

##### Release Branch

A branch has to be created for every release. The naming rule for this
branch is olingo-${version}. This has to be created
manually upon release approval.

### Release Candidate

Once all preparations are done, a release candidate will be built.

All release candidates must be cryptographically signed. The string
"-RCxx" will be attached to the version number of the release candidate
artifacts, where is the number of the release candidate starting with 01.
If more than one release candidate is required a new tag has to be created
and release candidate number will be increased by one.

The release candidate artifacts:

  - Maven artifacts will be staged on repository.apache.org. A new staging repo
is created per RC and will be communicated upon release.
  - Distribution commodity packages are staged at
http://people.apache.org/~[username]/olingo2/[version] (e.g. http://people.apache.org/~mibo/olingo2/2.0.0-RC01)

Once candidate artifacts are available, release manager kicks off the [VOTE process][3].

If the vote fails, the raised issues will be fixed, a new release candidate will be
built and the VOTE process will be restarted.

If the release candidate gets approved, we can proceed to release publishing.

#### How to verify a Release Candidate

This checklist helps verifying if a release candidate is valid:

* Are all files on "http://people.apache.org/~[username]/olingo2/[version]"?
* Check if md5, sha512 and asc files are filled correctly?
* Can the zip files be unpacked without issues?
* Execute a "mvn clean install -Pbuild.quality" on parent distribution. It should work without issues.
* Does the JavaDoc only contain API documentation?
* Is there a Disclaimer, Notice, License and Dependencies File in every folder?
* Do all License files contain the right amount of licenses?
* Do Notice files mention 3rd party libraries if they are contained in the distribution?

After all questions of this checklist can be answered with yes it is OK to give a +1 on the mailing list.
Of course the Release Manager can also use this checklist to make sure all artifacts are correct before publishing the results on the mailing list.


### Publishing the Release

If the release candidate gets approved, we can proceed to release publishing:

  - Release candidate maven artifacts are promoted in the Apache Maven Repository and
made available [here](https://repository.apache.org/index.html#nexus-search;gav~org.apache.olingo~~~~).
  - Publish final release Version to [Apache Repository](https://repository.apache.org/)
    - First publish via `mvn deploy -Papache-release` into the *Staging Area*
    - From *Staging Area* close and release the staged Artifacts to finish publishing
    - Afterwards the Maven artifacts are automatically synced to [Maven Central](http://search.maven.org/#search|ga|1|org.apache.olingo).
  - Release candidate commodity packages are synced (together with their checksum and
signatures) to [Apache Distributions](http://www.apache.org/dist/olingo/).
  - Release tag is renamed to final version.
  - Release branch is created.
  - Release is closed in Jira.
  - Release is announced to dev@olingo.apache.org, announce@apache.org.

### Maintain Release Distributions

To maintain the released Distributions for the download pages following steps are required (for more information see [Apache Distribution Documentation](http://www.apache.org/dev/release-publishing.html#distribution_dist)) to upload the new distribution via SVN (`svn co https://dist.apache.org/repos/dist/release/olingo`):

  - Check out latest SVN revision via `svn co https://dist.apache.org/repos/dist/release/olingo`
  - Change into the directory according to the released Olingo artifact (e.g. `odata2`, `odata4`, ...)
  - Create new directory according to release version (e.g `rel-x.x.x`) and copy all distribution artifacts (includes *.zip*, *.asc*, *.md5*, *.sha512*).
  - Add new directory (e.g. `svn add rel-x.x.x`) and do the commit (e.g `svn ci -m "Added Olingo x.x.x release"`)
  - Afterwards do a cleanup for the old releases according to the Apache Release Guidelines [When](http://www.apache.org/dev/release.html#when-to-archive) and [How](http://www.apache.org/dev/release.html#how-to-archive) to archive.


### Maintain Version Section in DOAP File

[http://olingo.apache.org/doap_Olingo.rdf][4]

Results are shown here:

[link text][5]

### Additional Apache Release Information

  - [Releases Policy][6]
  - [Publishing Releases][7]
  - [Publishing Maven Artifacts][8]


  [1]: http://olingo.apache.org/documentation.html
  [2]: https://issues.apache.org/jira/browse/OLINGO/fixforversion/12324804
  [4]: http://olingo.apache.org/doap_Olingo.rdf
  [5]: http://projects.apache.org/indexes/alpha.html#O
  [6]: http://www.apache.org/dev/release.html
  [7]: http://www.apache.org/dev/release-publishing.html
  [8]: http://www.apache.org/dev/publishing-maven-artifacts.html

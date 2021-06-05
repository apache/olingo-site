Title: Javascript Release Documentation

# Apache Olingo OData Client for JavaScript Release Documentation

---

### Introduction

This document describes the release guidelines for Apache Olingo OData Client for JavaScript (referenced in this document as ODataJS).
This is similar to [standard Apache procedures to release](http://maven.apache.org/developers/release/apache-release.html)
for Maven based projects at Apache, but since it is an JavaScript library it is adopted to be used with grunt js.

### Build Environment

Apache Olingo is built and released with [Node.js®](http://nodejs.org/) in combination with
[grunt task runner](http://gruntjs.com/)

### Release Artifacts

An Apache Olingo ODataJS release consists of:

   * **Main artifact**: `<artifactId>-<version>.<ext>`
   <br/> A library artifact bundle containing all libraries necessary use the OData Client for JavaScript
   <br/> **Package formats**: zip.

   * **JavaScript Doc artifact**: `<artifactId>-<version>-doc.<ext>`
   <br/> A documentation bundle containing the documentation for the ODataJS library.
   <br/> **Package formats**: zip.

   * **Source artifact**: `<artifactId>-<version>-sources.<ext>`
   <br/> A source-release bundle containing all files the sources necessary to build all other artifacts.
   <br/> **Package formats**: zip.


### Documentation and JavaDoc

The documentation that will be part of the release must match the code.
The documentation must be up-to-date. Release independend documentation is
maintained on the [Apache Olingo Documentation][1] page.

### Preparation

##### Release Manager

A release manager must be appointed for a release. He or she is in charge of the release process,
following the guidelines and eventually generating the release artifacts.
The release manager might tailor the process for a specific release.

##### Version

The Olingo community decides if the release will be a major or a minor release and
agrees on a version number.

The artifactnames are build according the following rule:

``artifactname`` : '${pkg.name}-${pkg.version}-${pkg.postfix}-${pkg.releaseCandidate}

*pkg* refers to the ``package.json`` file

*name* is the attribute ``name``

*version* is the attribute ``version``

*postfix* is the attribute ``postfix``

*isReleaseCandidate* is the attribute ``releaseCandidate``

package.json sample:

    {
      "name": "odatajs",
      "version": "4.0.0",
      "postfix": "",
      "releaseCandidate" : "RC01",
      ...
    }

Workflow for changing the release

    <change package.json ..., "name": "odatajs", "version": "4.0.0", ...

    <change package.json ... , "postfix": "", "releaseCandidate" : "RC01", ...
    git add package.json
    git commit -am 'Issue OLINGO-25 - make release - set version 4.0.0-RC01'
    git tag -f 4.0.0-RC01
    grunt release

    git push
    git push --tags

Release artifacts will be stored in the ``dist`` folder parallel to the ``olingojs`` folder.



##### Open Issues

There must not be any open JIRA issues for this release. There might be open issues for
future releases.

##### Unit Tests and Integration Tests

The QUnit tests in **odata-qunit-tests.htm** tests must succeed on a clean machine with the c#.net testserver running.

    Open http://localhost:4002/tests/odata-qunit-tests.htm in the browsers  recommented ( Internet Explorer, Chrome, Firefox and Opera)

##### Apache License and Code Style

Each source code file must have a current ASF license header. The source
code should follow the Apache Olingo code style. For verification run following
Maven execution

    ``grunt license-check``
    must be ok

##### Packaging

NOTICE, LICENSE and DISCLAIMER must be present in all bundles and must be up-to-date.

##### MD5 and SHA for distribution packages

MD5 and SHA files are created manually for distribution packages:

    openssl md5 < ${filename}.zip > ${filename}.zip.md5
    gpg --print-md SHA512 ${filename}.zip > ${filename}.zip.sha512


##### Release Tag

A tag has to be created for every release candidate. The naming rule
for the tags is olingojs-${version}-${postfix}-RCxx. This is created as
part of the Maven release process. The tag will be renamed to the
final version number upon vote approval.

##### Release Branch

A branch has to be created for every release. The naming rule for this
branch is olingojs-${version}-${postfix}. This has to be created
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
http://people.apache.org/~[username]/olingo4/js/[version] (e.g. http://people.apache.org/~mibo/olingo4/js/4.0.0-RC01)

Once candidate artifacts are available, release manager kicks off the [VOTE process][2].

If the vote fails, the raised issues will be fixed, a new release candidate will be
built and the VOTE process will be restarted.

If the release candidate gets approved, we can proceed to release publishing.

#### How to verify a Release Candidate

This checklist helps verifying if a release candidate is valid:

* Are all files on "http://people.apache.org/~[username]/olingo4/js/[version]"?
* Check if md5, sha512 and asc files are filled correctly?
* Can the zip files be unpacked without issues?
* Execute a "grunt build"
* Start the test server
* Is there a Disclaimer, Notice, License and Dependencies File in every folder?
* Do all License files contain the right amount of licenses?
* Do Notice files mention 3rd party libraries if they are contained in the distribution?

After all questions of this checklist can be answered with yes it is OK to give a +1 on the mailing list.
Of course the Release Manager can also use this checklist to make sure all artifacts are correct before publishing the results on the mailing list.


### Publishing the Release

If the release candidate gets approved, we can proceed to release publishing:

  - Publish final release Version to [Apache Repository](https://repository.apache.org/)
    - Copy the release Artifacts into the *Staging Area*
    - From *Staging Area* close and release the staged Artifacts to finish publishing
    - Afterwards the Maven artifacts are automatically synced to [Maven Central](http://search.maven.org/#search|ga|1|org.apache.olingo).
  - Release candidate commodity packages are synced (together with their checksum and
signatures) to [Apache Distributions](http://www.apache.org/dist/olingo/).
  - Release tag is renamed to final version.
  - Release branch is created.
  - Release is closed in Jira.
  - Release is announced to dev@olingo.apache.org, announce@apache.org.

### Maintain Version Section in DOAP File

[http://olingo.apache.org/doap_Olingo.rdf][3]

Results are shown here:

[TBD][4]

### Additional Apache Release Information

  - [Releases Policy][5]
  - [Publishing Releases][6]
  - [Publishing Maven Artifacts][7]


  [1]: /documentation.html
  [3]: /doap_Olingo.rdf
  [4]: http://projects.apache.org/indexes/alpha.html#O
  [5]: http://www.apache.org/dev/release.html
  [6]: http://www.apache.org/dev/release-publishing.html
  [7]: http://www.apache.org/dev/publishing-maven-artifacts.html

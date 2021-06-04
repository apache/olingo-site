Title:     Project build for the OData library for JavaScript

# Building the Olingo OData Client for JavaScript 

### Test tool prerequisites

Open your (git-)bash and navigate to the folder 
``olingo-odata4-js/``

Entry the *grunt -version* command to show the version of the globaly installed grund-client and the local grunt installation:

    $ grunt -version
    grunt-cli v0.1.13
    grunt v0.4.5

### Build

To build the odatajs library use the grunt task chain **build**.

**``grunt build``**

This task builds the odatajs library

The files are created in the ./_build folder:

* /doc-src
  <br/>Documentation
* /lib
  <br/>The  ODataJs library
* /tmp
  <br/>Folder for temporary build artefacts. E.g. logfile from the license-check

**``grunt release``**

This task builds the odatajs library and creates the files for the release

The release files are created in the ./_dist/odatajs-<version> folder:

* odatajs-version-doc.zip
  <br/>Documentation, "/doc" directory zipped to file **odatajs-version-doc.zip**
* odatajs-version-lib.zip
  <br/>Library, zipped "/lib" directory to file **odatajs-version-lib.zip**
* odatajs-version-sources.zip
  <br/>Sources, zipped "/sources" directory to file **odatajs-version-sources.zip**

**``release-sign``**

Signs the zipped files in the /_dist folder. The signing files are also stored in the /_dist folder.

### Sign the NuGet package

The odatajs.<version>.nupkg filed needs to be signed manually.

* md5 
  <br/>>openssl dgst -md5 odatajs.4.0.0.nupkg > odatajs.4.0.0.nupkg.md5*
* sha512
  <br/>>gpg --print-md SHA512 odatajs.4.0.0.nupkg > odatajs.4.0.0.nupkg.sha512
* asc
  <br/>>gpg --armor --detach-sign odatajs.4.0.0.nupkg


### Check the license headers

Please ensure that the rat tool is properly installed (see project-setup documentaion) and run

**``grunt license-check``**

The license-check log files are created in the ./_build/tmp directory.

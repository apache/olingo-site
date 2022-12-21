Title: How To Contribute

# Apache Olingo - How To Contribute

**Content**

[TOC]

### Overview

If you want to contribute to the Olingo Project, you can submit patches, report bugs or provide documentation and tutorials. Contributions are managed in the project [JIRA](https://issues.apache.org/jira/browse/OLINGO). So if you found a bug or want to provide a contribution please open a new [JIRA](https://issues.apache.org/jira/browse/OLINGO) ticket. Detailed information how to contribute can be found in the following chapter.

### Contribute via Git-Patch/Pull-Request and Olingo-JIRA

The Olingo uses the following process for contributions:

 1. Clone the Olingo Project
 2. Report a bug or feature (you must be [signed in](https://issues.apache.org/jira/login.jsp) to open an issue)
 3. Develop the according bugfix / feature
 4. Prepare your code for contribution
 5. Submit your JIRA ticket to receive feedback

### Report a bug or feature
If have you found a bug, please provide a detailed explanation how the bug can be reproduced.

You should mention the following properties:

  - Version of the library e.g. `(Java) V4 4.0.0`
  - If a part of the implementation is not compliant to the OData protocol, please link the related part of the speciation or provide a link to the related JIRA issue of OData protocol. The official OASIS-OData JIRA can be found [here](https://issues.oasis-open.org/browse/ODATA/)
  - Often bugs are related to the underlying metadata document. Please append the related part of the metadata document to reproduce the bug.
  - Provide a code snipped to reproduce the bug.
  - If you receive an exception, provide the stack trace of the exception.
  - Append further information in which part of the library the bug occurs (if available).
  - If you have fixed a bug, you can append the solution to the request. Further information how to append your solution to a ticket are provided in the following chapters.

**Bug/Feature Priorities**
The Olingo project uses the following definition of priorities:

  - *Blocker* - This bug/feature must be fixed/implemented before the next release
  - *Critical* - This bug/feature should be resolved/implemented immediately and must be fixed/implemented before the next major release.
  - *Major* - This bug/feature should be resolved/implemented as soon as possible in the normal course of development activity, before the software is released.
  - *Minor* (**default**) - This bug/feature should be fixed/implemented after serious bugs have been fixed/implemented.
  - *Trivial* - This bug/feature can be resolved in a future major system revision or not be resolved/implemented at all.

The default priority is **Minor**.

**Bug/Feature Types**
The Olingo project uses the following *types* for an issue:

  - *Feature* - An issue that introduced a new feature (behaviour) to the Olingo library (this could be e.g. an additional not yet implemented part of the OData specification or convenience methods)
  - *Improvement* - An issue that improves an existing feature (behaviour) of the Olingo library (this could be e.g. a convenience method or internal improvements which does not change behavior)
  - *Bug* - An issue that describes an incorrect behaviour of the Olingo library in relation to the according OData specification.
  - *Test* - An issue that only improve or extends tests for the  Olingo library.
  - *Task* - An issue that improve or extends basic parts for the Olingo library without changing code (or logic). This could be e.g. improvements or changes necessary for the build infrastructure.
  - *Question* - An issue that only is a question related to the Olingo library.


### Develop the according bugfix / feature

#### Clone the Olingo Project
The current development version can be found in the Apache git repository.
Please note Olingo provides two different libraries. One one hand Olingo V2 which implements the OData V2 specification and Olingo V4, which implements the OData V4 specification.

To clone the current master branch of the Olingo project please use one of the following commands:

**Olingo V2 Code**

    git clone https://gitbox.apache.org/repos/asf/olingo-odata2

**Olingo V4 code**

    git clone https://gitbox.apache.org/repos/asf/olingo-odata4


#### Providing bugfix / code
To provide a bug fix, checkout the current master branch of the project and develop your solution. In Olingo we truly believe in tests, so your contribution should at least contain tests that show that your contribution works as expected and also tests that reproduces the previous reported bug. If you provide a new feature your tests should reach at least 80 percent test-coverage.

To ensure this we have lots of tests and use [Cobertura](https://cobertura.github.io/cobertura/) as *code coverage tool*. In addition there exists separate build jobs on the Apache Build servers for latest versions of [Olingo V2](https://builds.apache.org/job/olingo-odata2-cobertura/) and [Olingo V4](https://builds.apache.org/job/olingo-odata4-cobertura/).

To append your contribution to a JIRA ticket, please create a patch file as explained in the chapter.

**Providing documentation for the Apache website**
To provide documentation or tutorials you should write your contribution using [a Markdown syntax](www.apache.org/dev/cmsref#markdown).


#### Prepare your code for contribution

To append you contribution to a JIRA issue, please create a [patch file](https://git-scm.com/docs/git-format-patch). The commit message should contain the JIRA request number (e.g.  OLINGO-42) and a short commit message.

**Example - Create a patch file**
You have done several commits and want to provide a single commit which contains all your changes.

	...
	git commit -m "[OLINGO-42] Start development new feature"
    git commit -m "[OLINGO-42] Added new tests"
	git commit -m "[OLINGO-42] Typo fixed"
	...
	git rebase -i HEAD~3
	git format-patch -1

First rebase your local history to create a single commit. After that create a patch file using `git format-patch`. Now you can upload and append your created patch file to the JIRA issue.

**Example - Applying a patch file**
You like to apply a patch file named "OLINGO_42.patch". Use the following commands:

    git am --signoff OLINGO_42.patch

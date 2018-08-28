
## Overview

This is an assessment of the Pulsar podling’s maturity, meant to help inform the decision (of the mentors, community, Incubator PMC and ASF Board of Directors) to graduate it as a top-level Apache project.

It is based on the ASF project maturity model at https://community.apache.org/apache-way/apache-project-maturity-model.html


## Status of this document

All open items are updated with the latest status.

## Maturity model assessment

Mentors and community members are encouraged to contribute to this page and comment on it, the following table summarizes project’s self-assessment against the Apache Maturity Model.


| ID | Description | Status |
|:---|:------------|:-------|
| ***Code*** | | |
| CD10     | The project produces Open Source software, for distribution to the public at no charge. | **YES** The project source code is licensed under the Apache License, version 2.0.                      |
| CD20     | The project's code is easily discoverable and publicly accessible. | **YES** Linked from the website, available via GitBox  https://gitbox.apache.org/repos/asf?p=incubator-pulsar.git and https://github.com/apache/incubator-pulsar. |
| CD30     | The code can be built in a reproducible way using widely available standard tools. | **YES** The build uses Apache Maven for Java code and CMake for C++ code. Continuous integration is used to automate the testing and validation of new commits. |
| CD40     | The full history of the project's code is available via a source code control system, in a way that allows any released version to be recreated. | **YES**  All the history of the project is available through Git. All releases are properly tagged. |
| CD50     | The provenance of each line of code is established via the source code control system, in a reliable way based on strong authentication of the committer. When third-party contributions are committed, commit messages provide reliable information about the code provenance.  | **YES** The git repository is managed by Apache Infra. Only Pulsar committers have write access. All code is checked in after at least 2 committers have reviewed and approved a pull-request. For 3rd party contribution, the commit message and logs will include all the details of author and committer. |
| ***Licenses and Copyright*** | | |
| LC10     | The code is released under the Apache License, version 2.0. | **YES** Source distributions clearly state license. Convenience binaries clearly state license. |
| LC20     | Libraries that are mandatory dependencies of the project's code do not create more restrictions than the Apache License does. | **YES** The list of mandatory dependencies have been reviewed to contain approved licenses only.|
| LC30     | The libraries mentioned in LC20 are available as Open Source software. | **YES** All dependencies are available as open source software. |
| LC40     | Committers are bound by an Individual Contributor Agreement (the "Apache iCLA") that defines which code they are allowed to commit and how they need to identify code that is not their own. | **YES** The project uses a repository managed by Apache Infra &mdash; write access requires an Apache account, which requires an ICLA on file. |
| LC50     | The copyright ownership of everything that the project produces is clearly defined and documented. | **YES** Software Grant Agreement for the initial donation from Yahoo was filed. All files in the source repository have appropriate headers. Automated process is in place to ensure every file has expected headers. |
| ***Releases*** | | |
| RE10     | Releases consist of source code, distributed using standard and open archive formats that are expected to stay readable in the long term. | **YES** Source releases are distributed via https://dist.apache.org/repos/dist/release/incubator/pulsar/ and linked from the website at https://pulsar.incubator.apache.org/download/.
| RE20     | Releases are approved by the project's PMC (see CS10), in order to make them an act of the Foundation.| **YES** All incubating releases have been approved by the Pulsar community with at least 3 PPMC votes and from the Incubator with 3 IPMC votes. |
| RE30     | Releases are signed and/or distributed along with digests that can be reliably used to validate the downloaded archives. | **YES** All releases are signed, and the KEYS file is provided on dist.apache.org. |
| RE40     | Convenience binaries can be distributed alongside source code but they are not Apache Releases -- they are just a convenience provided with no guarantee. | **YES** Convenience binaries are distributed via via dist.apache.org. Java binary artifacts are also distributed through Maven Central Repository. |
| RE50     | The release process is documented and repeatable to the extent that someone new to the project is able to independently generate the complete set of artifacts required for a release. | **YES** Step-by-step release guide is available describing the entire process. The Pulsar releases have been performed by 5 different release managers. |
| ***Quality*** | | |
| QU10     | The project is open and honest about the quality of its code. Various levels of quality and maturity for various modules are natural and acceptable as long as they are clearly communicated. | **YES** The project records all bugs in the GitHub issue tracker at https://github.com/apache/incubator-pulsar/issues |
| QU20     | The project puts a very high priority on producing secure software. | **YES** Security issues are treated with the highest priority, according to the CVE/Security Advisory procedure. |
| QU30     | The project provides a well-documented channel to report security issues, along with a documented way of responding to them. | **YES**  Website provides a link to ASF security page https://www.apache.org/security/ from each page. |
| QU40     | The project puts a high priority on backwards compatibility and aims to document any incompatible changes and provide tools and documentation to help users transition to new features. | **YES** Pulsar releases have to goal not to break backward and forward compatibility across releases, even across major ones. Regarding APIs, methods are marked as deprecated and warning is provided that at some point in future they might be removed. Each release contains a summary of most important changes and fixed, combined with the full list of issues fixed from Github |
| QU50     | The project strives to respond to documented bug reports in a timely manner. | **YES** The project has resolved 378 bugs during incubation. |
| ***Community*** | | |
| CO10     | The project has a well-known homepage that points to all the information required to operate according to this maturity model. | **YES** The project website has a description of the project with technical details, how to contribute, team. |
| CO20     | The community welcomes contributions from anyone who acts in good faith and in a respectful manner and adds value to the project. | **YES** It’s part of the contribution guide (http://pulsar.incubator.apache.org/contributing) and the current committers are really keen to welcome contributions.
| CO30     | Contributions include not only source code, but also documentation, constructive bug reports, constructive discussions, marketing and generally anything that adds value to the project. | **YES** The contribution guide refers to non source code contribution, like documentation. |
| CO40     | The community is meritocratic and over time aims to give more rights and responsibilities to contributors who add value to the project. | **YES** The community has elected 5 new committers during incubation, based on meritocracy. |
| CO50     | The way in which contributors can be granted more rights such as commit access or decision power is clearly documented and is the same for all contributors. | **YES** The criteria is documented in the contribution guide. |
| CO60     | The community operates based on consensus of its members (see CS10) who have decision power. Dictators, benevolent or not, are not welcome in Apache projects. | **YES** The project works to build consensus. All votes from the community have been unanimous so far. All technical discussions and disagreements have been resolved in a positive ways, addressing the concerns from all the people involved.
| CO70     | The project strives to answer user questions in a timely manner. | **YES** The project typically provides detailed answers to user questions within a few hours via dev@ and user@ mailing lists.
| ***Consensus Building*** | | |
| CS10     | The project maintains a public list of its contributors who have decision power -- the project's PMC (Project Management Committee) consists of those contributors. | **YES** The website contains the list of committers and PPMC members at http://pulsar.incubator.apache.org/team/.
| CS20     | Decisions are made by consensus among PMC members and are documented on the project's main communications channel. Community opinions are taken into account but the PMC has the final word if needed. | **YES** The project has been making all decisions on the project mailing lists. |
| CS30     | Documented voting rules are used to build consensus when discussion is not sufficient. | **YES** The project uses the standard ASF voting rules. Voting rules are clearly stated before the voting starts for each individual vote. |
| CS40     | In Apache projects, vetoes are only valid for code commits and are justified by a technical explanation, as per the Apache voting rules defined in CS30. | **YES** The project hasn’t used a veto at any point and relies on mandatory code reviews. |
| CS50     | All "important" discussions happen asynchronously in written form on the project's main communications channel. Offline, face-to-face or private discussions that affect the project are also documented on that channel. | **YES** The project has been making all decisions on the project mailing lists. Technical discussions may happen during code reviews, or when commenting on issues. These conversations are also in written form and asynchronous, and are copied back to the project lists for archival.


This page contains instructions for Pulsar committers on
how to perform a release.

If you haven't already did it. Create and publish the GPG
key with which you'll be signing the release artifacts.
Instructions are at [Create GPG keys to sign release artifacts](https://github.com/apache/pulsar/wiki/Create-GPG-keys-to-sign-release-artifacts).

## Making the release

The steps for releasing are as follows:
1. Create release branch
2. Update project version and tag
3. Build and inspect the artifacts
4. Inspect the artifacts
5. Stage artifacts in maven
6. Move master branch to next version
7. Write release notes
8. Run the vote
10. Promote the release
11. Publish Docker Images
12. Publish Python Clients
13. Update release notes
14. Update the site
15. Announce the release
16. Write a blog post for the release (optional)

## Steps in detail

#### 1. Create the release branch

We are going to create a branch from `master` to `branch-v2.X`
where the tag will be generated and where new fixes will be
applied as part of the maintenance for the release.

The branch needs only to be created when creating major releases,
and not for patch releases.

Eg: When creating `v2.3.0` release, will be creating
the branch `branch-2.3`, but for for `v2.3.1` we
would keep using the old `branch-2.3`.

In these instructions, I'm referring to an fictitious release `2.X.0`. Change the release version in the examples
accordingly with the real version.

It is recommended to create a fresh clone of the repository to avoid any local files to interfere
in the process:

```shell
git clone git@github.com:apache/pulsar.git
cd pulsar
git checkout -b branch-2.X origin/master
```

#### 2. Update project version and tag

During the release process, we are going to initially create
"candidate" tags, that after verification and approval will
get promoted to the "real" final tag.

In this process the maven version of the project will always
be the final one.

```shell
# Bump to the release version
mvn versions:set -DnewVersion=2.X.0
mvn versions:set -DnewVersion=2.X.0 -pl buildtools
mvn versions:set -DnewVersion=2.X.0 -pl pulsar-sql/presto-distribution

# Commit
git commit -m 'Release 2.X.0' -a

# Create a "candidate" tag
git tag -u $USER@apache.org v2.X.0-candidate-1 -m 'Release v1.X.0-candidate-1'

# Push both the branch and the tag to Github repo
git push origin branch-2.X
git push origin v2.X.0-candidate-1
```

#### 3. Build and inspect the artifacts

```shell
mvn install
```

After the build, there will be 4 generated artifacts:

 * `distribution/server/target/apache-pulsar-2.X.0-bin.tar.gz`
 * `distribution/server/target/apache-pulsar-2.X.0-src.tar.gz`
 * `distribution/io/target/apache-pulsar-io-connectors-2.X.0-bin.tar.gz`
 * `distribution/offloaders/target/apache-pulsar-offloaders-2.X.0-bin.tar.gz`

Inspect the artifacts:
 * Unpack both of them
 * Check that the `LICENSE` and `NOTICE` files cover all included jars (especially for the -bin package)
   - Use script to cross-validate `LICENSE` file with included jars:
      ```
      src/check-binary-license distribution/server/target/apache-pulsar-2.x.0-bin.tar.gz
      ```
 * Run Apache RAT to verify the license headers in the `src` package:
 ```shell
 mvn apache-rat:check
 ```
 * Check that the standalone Pulsar service starts correctly:
 ```shell
 bin/pulsar standalone
 ```

* Use instructions in [Release-Candidate-Validation](Release-Candidate-Validation) to do some sanity checks on the produced binary distributions.

##### 3.1. Build RPM and DEB packages

```shell
pulsar-client-cpp/pkg/rpm/docker-build-rpm.sh

pulsar-client-cpp/pkg/deb/docker-build-deb.sh
```

This will leave the RPM/YUM and DEB repo files in `pulsar-client-cpp/pkg/rpm/RPMS/x86_64` and
`pulsar-client-cpp/pkg/deb/BUILD/DEB` directory.

#### 4. Sign and stage the artifacts

The `src` and `bin` artifacts need to be signed and uploaded to the dist SVN
repository for staging.

```shell
svn co https://dist.apache.org/repos/dist/dev/pulsar pulsar-dist-dev
cd svn pulsar-dist-dev

# '-candidate-1' needs to be incremented in case of multiple iteration in getting
#    to the final release)
svn mkdir pulsar-2.X.0-candidate-1

cd pulsar-2.X.0-candidate-1
$PULSAR_PATH/src/stage-release.sh .

svn add *
svn ci -m 'Staging artifacts and signature for Pulsar release 2.X.0'
```

#### 5. Stage artifacts in maven

Upload the artifacts to ASF Nexus:

```shell
export APACHE_USER=$USER
export APACHE_PASSWORD=$MY_PASSWORD
mvn deploy -DskipTests -Papache-release --settings .travis/settings.xml
```

This will ask for the GPG key passphrase and then upload to the staging repository.

Login to ASF Nexus repository at https://repository.apache.org

Click on "Staging Repositories" on the left sidebar and then select the current
Pulsar staging repo. This should be called something like `orgapachepulsar-XYZ`.

Use the "Close" button to close the repository. This operation will take few
minutes. Once complete click "Refresh" and now a link to the staging repository
should be available, something like
https://repository.apache.org/content/repositories/orgapachepulsar-XYZ


#### 6. Move master branch to next version

We need to move master version to next iteration `X + 1`.

```
git checkout master
mvn versions:set -DnewVersion=2.Y.0-SNAPSHOT
mvn versions:set -DnewVersion=2.Y.0-SNAPSHOT -pl buildtools
mvn versions:set -DnewVersion=2.Y.0-SNAPSHOT -pl pulsar-sql/presto-distribution

git commit -m 'Bumped version to 2.Y.0-SNAPSHOT' -a
```

Since this needs to be merged in `master`, we need to follow the regular process
and create a Pull Request on github.

#### 7. Write release notes

Check the milestone in Github associated with the release.
https://github.com/apache/pulsar/milestones?closed=1

In the release item, add the list of most important changes that happened in the
release and a link to the associated milestone, with the complete list of all the
changes.

#### 8. Run the vote

Send an email on the Pulsar Dev mailing list:

```
To: dev@pulsar.apache.org
Subject: [VOTE] Pulsar Release 2.X.0 Candidate 1

This is the first release candidate for Apache Pulsar, version 2.X.0.

It fixes the following issues:
https://github.com/apache/pulsar/milestone/8?closed=1

*** Please download, test and vote on this release. This vote will stay open
for at least 72 hours ***

Note that we are voting upon the source (tag), binaries are provided for
convenience.

Source and binary files:
https://dist.apache.org/repos/dist/dev/pulsar/pulsar-2.X.0-candidate-1/

SHA-1 checksums:

028313cbbb24c5647e85a6df58a48d3c560aacc9  apache-pulsar-2.X.0-SNAPSHOT-bin.tar.gz
f7cc55137281d5257e3c8127e1bc7016408834b1  apache-pulsar-2.x.0-SNAPSHOT-src.tar.gz

Maven staging repo:
https://repository.apache.org/content/repositories/orgapachepulsar-169/

The tag to be voted upon:
v2.X.0-candidate-1 (21f4a4cffefaa9391b79d79a7849da9c539af834)
https://github.com/apache/pulsar/releases/tag/v2.X.0-candidate-1

Pulsar's KEYS file containing PGP keys we use to sign the release:
https://dist.apache.org/repos/dist/release/pulsar/KEYS

Please download the the source package, and follow the README to build
and run the Pulsar standalone service.
```

The vote should be open for at least 72 hours (3 days). Votes from Pulsar PMC members
will be considered binding, while anyone else is encouraged to verify the release and
vote as well.

If the release is approved here, we can then proceed to next step.

#### 10. Promote the release

Create the final git tag:

```shell
git tag -u $USER@apache.org v2.X.0 -m 'Release v2.X.0'
git push origin v2.X.0
```

Promote the artifacts on the release location:
```shell
svn move https://dist.apache.org/repos/dist/dev/pulsar/pulsar-2.X.0-candidate-1 \
         https://dist.apache.org/repos/dist/release/pulsar/pulsar-2.X.0
```

Remove the old releases (if any). We only need the latest release there, older releases are
available through the Apache archive:

```shell
# Get the list of releases
svn ls https://dist.apache.org/repos/dist/release/pulsar

# Delete each release (except for the last one)
svn rm https://dist.apache.org/repos/dist/release/pulsar/pulsar-2.Y.0
```

Promote the Maven staging repository for release. Login to `https://repository.apache.org` and
select the staging repository associated with the RC candidate that was approved. The naming
will be like `orgapachepulsar-XYZ`. Select the repository and click on "Release". Artifacts
will now be made available on Maven central.

#### 11. Publish Docker Images

- a) Trigger the [pulsar-release-binaries](https://builds.apache.org/job/pulsar-release-binaries/) with the tag name.
- b) once it is done, check https://hub.docker.com/r/apachepulsar/pulsar/tags/ to make sure the docker image is published.
- c) try to run `docker pull apachepulsar/pulsar:<tag>` to fetch the image, run the docker in standalone mode and make sure the docker image is okay.

#### 12. Publish Python Clients

> ##### NOTES:
>
> 1. you need to create an account on PyPI: https://pypi.org/project/pulsar-client/
>
> 2. ask Matteo or Sijie for adding you as a maintainer for pulsar-client on PyPI

##### Linux

There is a script that builds and packages the Python client inside Docker images.

> make sure you run following command at the release tag!!

```shell
$ pulsar-client-cpp/docker/build-wheels.sh
```

The wheel files will be left under `pulsar-client-cpp/python/wheelhouse`. Make sure all the files has `manylinux` in the filenames. Otherwise those files will not be able to upload to PyPI.

Run following command to push the built wheel files.

```shell
$ cd pulsar-client-cpp/python/wheelhouse

$ twine upload pulsar_client-*.whl
```

##### MacOS

> ###### NOTES
>
> You need to install following softwares before proceeding:
>
> - [VirtualBox](https://www.virtualbox.org/)
> - [VirtualBox Extension Pack]
> - [Vagrant](https://www.vagrantup.com/)
> - [Vagrant-scp](https://github.com/invernizzi/vagrant-scp)
>
> And make sure your laptop have enough disk spaces (> 30GB) since the build scripts
> will download MacOS images, launch them in VirtualBox and build the python
> scripts.

Build the python scripts.

```shell
$ cd pulsar-client-cpp/python/pkg/osx/
$ ./generate-all-wheel.sh
```

The wheel files will be generated at each platform directory under `pulsar-client-cpp/python/pkg/osx/`.
Then you can run `twin upload` to upload those wheel files.

If you don't have enough spaces, you can build the python wheel file platform by platform and remove the images under `~/.vagrant.d/boxes` between each run.

#### 13. Update release notes

Follow [this example](https://github.com/apache/pulsar/pull/2292) to add the release notes there.

#### 14. Update the site

> the website update always happen under `master` branch.

1. Create a new branch off master

```shell
git checkout -b doc_release_<release-version>
```

2. Go to the website directory

```shell
cd site2/website
```

3. Cut a new version of the documentation.

```shell
yarn run version <release-version>
```

After this command, a directory `version-<release-version>` will be added under `site2/website/versioned_docs`. Then,

```shell
git add versioned_docs/version-<release-version>
```

4. Update `releases.json` file to add `<release-version>` to the top of the list.

5. Update `Feature Matrix` section the the clients page `versioned_docs/version-<release-version>/getting-started-clients.md` to make it in-sync with the features available in the released version.

6. Send out a PR request for review.

7. Once the PR is approved and merged to master, trigger the [pulsar-website-build](https://builds.apache.org/job/pulsar-website-build/).

8. Once the build is done, the new website should be automatically published. Open https://pulsar.apache.org in your browsers to verify all the changes are alive.

#### 15. Announce the release

Once the release artifacts are available in the Apache Mirrors and the website is updated,
we need to announce the release.

Send an email on these lines:

```
To: dev@pulsar.apache.org, user@pulsar.apache.org, announce@apache.org
Subject: [ANNOUNCE] Apache Pulsar 2.X.0 released

The Apache Pulsar team is proud to announce Apache Pulsar version 2.X.0.

Pulsar is a highly scalable, low latency messaging platform running on
commodity hardware. It provides simple pub-sub semantics over topics,
guaranteed at-least-once delivery of messages, automatic cursor management for
subscribers, and cross-datacenter replication.

For Pulsar release details and downloads, visit:

https://pulsar.apache.org/download

Release Notes are at:
http://pulsar.apache.org/release-notes

We would like to thank the contributors that made the release possible.

Regards,

The Pulsar Team
```

16. Write a blog post for the release (optional)

It is encouraged to write a blog post to summarize the features introduced in this release,
especially for feature releases.
You can follow the example [here](https://github.com/apache/pulsar/pull/2308)

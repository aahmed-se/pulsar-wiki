
This page contains instructions for Pulsar committers on
how to perform a release.

If you haven't already did it. Create and publish the GPG
key with which you'll be signing the release artifacts.
Instructions are at [Create GPG keys to sign release artifacts](https://github.com/apache/incubator-pulsar/wiki/Create-GPG-keys-to-sign-release-artifacts).

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
9. Run the vote on Incubator
10. Promote the release
11. Update the site
12. Announce the release

## Steps in detail

#### 1. Create the release branch

We are going to create a branch from `master` to `branch-v1.X`
where the tag will be generated and where new fixes will be
applied as part of the maintenance for the release.

The branch needs only to be created when creating major releases,
and not for patch releases.

Eg: When creating `v1.19.0-incubating` release, will be creating
the branch `branch-1.19`, but for for `v1.19.1-incubating` we
would keep using the old `branch-1.19`.

In these instructions, I'm referring to an fictitious release `1.X.0-incubating`. Change the release version in the examples
accordingly with the real version.

It is recommended to create a fresh clone of the repository to avoid any local files to interfere
in the process:

```shell
git clone git@github.com:apache/incubator-pulsar.git
cd incubator-pulsar
git checkout -b branch-1.X origin/master
```

#### 2. Update project version and tag

During the release process, we are going to initially create
"candidate" tags, that after verification and approval will
get promoted to the "real" final tag.

In this process the maven version of the project will always
be the final one.

```shell
# Bump to the release version
mvn versions:set -DnewVersion=1.X.0-incubating

# Commit
git commit -m 'Release 1.X.0-incubating' -a

# Create a "candidate" tag
git tag -u $USER@apache.org v1.X.0-incubating-candidate-0 -m 'Release v1.X.0-incubating-candidate-0'

# Push both the branch and the tag to Github repo
git push origin branch-1.X
git push origin v1.X.0-incubating-candidate-0
```

#### 3. Build and inspect the artifacts

Perform the build in a Linux machine (that's because the build is compiling some C source files
for the `pulsar-checksum` module).

```shell
mvn install
```

After the build, there will be 2 generated artifacts:

 * `all/target/apache-pulsar-1.X.0-incubating-bin.tar.gz`
 * `all/target/apache-pulsar-1.X.0-incubating-src.tar.gz`

Inspect the artifacts:
 * Unpack both of them
 * Check that the `LICENSE` and `NOTICE` files cover all included jars (especially for the -bin package)
 * Run Apache RAT to verify the license headers in the `src` package:
 ```shell
 mvn apache-rat:check
 ```
 * Check that the standalone Pulsar service starts correctly:
 ```shell
 bin/pulsar standalone
 ```

#### 4. Sign and stage the artifacts

The `src` and `bin` artifacts need to be signed and uploaded to the dist SVN
repository for staging.

```shell
svn co https://dist.apache.org/repos/dist/dev/incubator/pulsar pulsar-dist-dev
cd svn pulsar-dist-dev

# '-candidate-0' needs to be incremented in case of multiple iteration in getting
#    to the final release)
svn mkdir pulsar-1.X.0-incubating-candidate-0

cd pulsar-1.X.0-incubating-candidate-0
cp /path/to/apachepulsar-1.X.0-incubating-src.tar.gz .
cp /path/to/apachepulsar-1.X.0-incubating-bin.tar.gz .

# There is a script to perform the signing in Pulsar repo
$PULSAR_PATH/src/sign-release.sh *.tar.gz

svn add *
svn ci -m 'Staging artifacts and signature for Pulsar release 1.X.0-incubating'
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
mvn versions:set -DnewVersion=1.Y.0-incubating-SNAPSHOT

git commit -m 'Bumped version to 1.Y.0-incubating-SNAPSHOT' -a
```

Since this needs to be merged in `master`, we need to follow the regular process
and create a Pull Request on github.

#### 7. Write release notes

Check the milestone in Github associated with the release.
https://github.com/apache/incubator-pulsar/milestones

In the release item, add the list of most important changes that happened in the
release and a link to the associated milestone, with the complete list of all the
changes.

#### 8. Run the vote

Send an email on the Pulsar Dev mailing list:

```
To: dev@pulsar.incubator.apache.org
Subject: [VOTE] Pulsar 1.X.0-incubating Release Candidate 0

This is the first release candidate for Apache Pulsar, version 1.X.0-incubating.

It fixes the following issues:
https://github.com/apache/incubator-pulsar/milestone/8

*** Please download, test and vote by July 29th 2017, 10:00 GMT.

Note that we are voting upon the source (tag), binaries are provided for
convenience.

Source and binary files:
https://dist.apache.org/repos/dist/dev/incubator/pulsar/pulsar-1.X.0-incubating-candidate-0/

Maven staging repo:
https://repository.apache.org/content/repositories/orgapachepulsar-169/

The tag to be voted upon:
v1.X.0-incubating-candidate-0 (21f4a4cffefaa9391b79d79a7849da9c539af834)

Pulsar's KEYS file containing PGP keys we use to sign the release:
https://dist.apache.org/repos/dist/release/incubator/pulsar/KEYS

Please download the the source package, and follow the README to build
and run the Pulsar standalone service.
```

The vote should be open for at least 72 hours (3 days). Votes from Pulsar PPMC members
will be considered binding, while anyone else is encouraged to verify the release and
vote as well.

If the release is approved here, we can then proceed to next step.

#### 9. Run the vote on Incubator

Since Pulsar is an incubator project, the release must be approved by the ASF Incubator PMC.

Start a `VOTE` thread on the incubator mailing list:


```
To: general@incubator.apache.org
Subject: [VOTE] Pulsar 1.X.0-incubating Release Candidate 0
....
```

As before, the vote should be open for at least 72 hours and it should get at least 3 binding +1s
from Incubator PMC members.

If the outcome is successful, we can continue on the next step, otherwise we'll fix the issues
and restart from step 2, this time issuing a `1.X.0-incubating-candidate-1` release.

#### 10. Promote the release

Create the final git tag:

```shell
git tag -u $USER@apache.org v1.X.0-incubating -m 'Release v1.X.0-incubating'
git push origin v1.X.0-incubating
```

Promote the artifacts on the release location:
```shell
svn move https://dist.apache.org/repos/dist/dev/incubator/pulsar/pulsar-1.X.0-candidate-0 \
         https://dist.apache.org/repos/dist/release/incubator/pulsar/pulsar-1.X.0
```

Remove the old releases (if any). We only need the latest release there, older releases are
available through the Apache archive:

```shell
# Get the list of releases
svn ls https://dist.apache.org/repos/dist/release/incubator/pulsar

# Delete each release (except for the last one)
svn rm https://dist.apache.org/repos/dist/release/incubator/pulsar/pulsar-1.Y.0
```

#### 11. Update the site

Copy the `latest` documentation into a version specific folder.

```shell
git checkout -b asf-site origin/asf-site
cd content/docs
cp -r latest v1.X.0-incubating
git add v1.X.0-incubating
git commit -a -m 'Copying generated documenation for v1.X.0-incubating'
git push origin asf-site
```

In `master` branch, update the site configuration to add the new version.

In `site/_config.yml` :

```yaml
# ...
current_version: 1.X.0-incubating
archived_releases:
    - 1.19.0-incubating

# ...
```

Commit the config change and submit a PR on Github.

#### 12. Announce the release

Once the release artifacts are available in the Apache Mirrors and the website is updated,
we need to announce the release.

Send an email on these lines:

```
To: dev@pulsar.incubator.apache.org, user@pulsar.incubator.apache.org, announce@apache.org
Subject: [ANNOUNCE] Apache Pulsar 1.X.0-incubating released

The Apache Pulsar team is proud to announce Apache Pulsar version 1.X.0-incubating.

This is the first Pulsar release after entering the Apache Incubator.

Pulsar is a highly scalable, low latency messaging platform running on
commodity hardware. It provides simple pub-sub semantics over topics,
guaranteed at-least-once delivery of messages, automatic cursor management for
subscribers, and cross-datacenter replication.

For Pulsar release details and downloads, visit:

https://pulsar.incubator.apache.org/download

Release Notes are at:
https://github.com/apache/incubator-pulsar/releases/tag/v1.X.0-incubating

We would like to thank the contributors that made the release possible.

Regards,

The Pulsar Team

DISCLAIMER:
Apache Pulsar is an effort undergoing incubation at The Apache Software
Foundation (ASF), sponsored by the Apache Incubator PMC. Incubation is
required of all newly accepted projects until a further review indicates that
the infrastructure, communications, and decision making process have
stabilized in a manner consistent with other successful ASF projects. While
incubation status is not necessarily a reflection of the completeness or stability
of the code, it does indicate that the project has yet to be fully endorsed by the ASF.
```

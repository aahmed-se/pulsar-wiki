
 * **Status**: Proposal
 * **Author**: Ali Ahmed
 * **Pull Request**: 
 * **Mailing List discussion**:

## Motivation

Currently the apache pulsar docker image is using jdk8 and python 2.7 as defaults, they are both EOL. The hope is we can move to more recent versions that are actively supported.

## Proposal and changes

I propose changing the Docker files to move the base image to use python3.5 and jdk11 as default env. Thin will mean calling java with call the java 11 runtime and python will linked to python3.5.

We are not building a custom modular jdk, simplu will target using the stock openjdk11 image.

There will some code minor changes and jar updates needed for the integration test to pass. The code will target java 1.8 runtime.

Python should not be affected but dependend systems like postgres for the dashboard will possibly be.

The target is to get these changes in for the 2.4 release.
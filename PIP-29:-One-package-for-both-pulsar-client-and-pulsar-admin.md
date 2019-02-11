* Status: **discussion**
* Author: [Sijie Guo](https://github.com/sijie)
* Pull Request: 
* Mailing List Discussion: 
* Release: N/A

## Motivation

Currently we are shipping `pulsar-client` and `pulsar-client-admin` separately.
Both `pulsar-client` and `pulsar-client-admin` are shaded packages. But they shaded
the dependencies independently.

It is quite common to see applications using both `pulsar-client` and `pulsar-client-admin`.
These applications will have redundant shaded classes existed in both `pulsar-client` and `pulsar-client-admin`.
Sometime it also causes troubles when we introduced new dependencies but forget to update shading rules.

## Proposal

This proposal is proposing introduced a new module called `pulsar-client-all`.
It will include both pulsar-client and pulsar-client-admin and shade the dependencies only one time.
This would reduce the size of dependencies for applications using both `pulsar-client` and `pulsar-client-admin`.

## Alternatives

There is no other alternatives. We can just leave as it is and do nothing.
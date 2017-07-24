
This page provides instructions for Pulsar committers on how to do
the initial gpg setup.

This is a condensed version of instructions available at
http://apache.org/dev/openpgp.html.


Install GnuPG. For example on MacOS:

```shell
brew install gnupg
```

Set configuration to use `SHA512` keys by default.

```shell
mkdir ~/.gnupg
echo <<< EOL
personal-digest-preferences SHA512
cert-digest-algo SHA512
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
EOL >> ~/.gnupg/gnupg.conf
```

The GPG key needs to be appended to `KEYS` file that is stored in 2 SVN locations,
one for proper releases and one for the release candidates.

The credentials for SVN are the usual Apache account credentials.

```shell
# Checkout the SNV folder containing the KEYS file
svn co https://dist.apache.org/repos/dist/dev/incubator/pulsar pulsar-dist-dev
cd pulsar-dist-dev

# Export the key in ascii format and append it to the file
( gpg --list-sigs $USER@apache.org
  gpg --export --armor $USER@apache.org ) >> KEYS

# Commit to SVN
svn ci -m "Added gpg key for $USER"
```

Repeat the same operation for the release KEYS file:

```shell
svn co https://dist.apache.org/repos/dist/release/incubator/pulsar pulsar-dist-release
cd pulsar-dist-release
# ... Same as above

( gpg --list-sigs $USER@apache.org
  gpg --export --armor $USER@apache.org ) >> KEYS

# Commit to SVN
svn ci -m "Added gpg key for $USER"
```

#### Upload the key to a public key server

Use the key id to publish it to a public key server:
```shell
gpg --send-key 8C75C738C33372AE198FD10CC238A8CAAC055FD2
```
# apt-release-check

This repository contains a single script `apt-release-check` which is a
Nagios/NRPE-compatible script for checking the integrity of the `Release` file
in a Debian APT repository.

If the process for indexing an APT repository fails for some reason, any hosts
configured to use the repository will fail on their next `apt update` with an
error like this:

    Failed to fetch http://repository.server.name/repo-name/dists/ ... directory/binary-amd64/Packages Hash Sum mismatch

This can break automated deployments and unattended updates, so monitoring the
index on the repository server may help to reduce collateral damage.

## How to use the script

Copy the script onto the repository server (e.g.: into `/usr/local/sbin`) and
then add a check entry to your NRPE config like this:

    command[check_apt_index]=/usr/local/sbin/apt-release-check /home/ftp/pub/apt-repo/debian/dists/stable/Release

It is also possible to check mulitple release files in one go using a wildcard:

    command[check_apt_index]=/usr/local/sbin/apt-release-check /home/ftp/pub/apt-repo/debian/dists/*/Release

## How it works

The script loops through checking each `Release` file named on the
command-line.  A `Release` file will contain a list of (relative) pathnames to
`Packages` files for each architecture, and hashes (MD5, SHA1, ...) for each of
these files.  The script confirms that each `Packages` file exists and that its
contents match each of the listed hashes.

Once the `Packages` hashes have been validated, the script will also attempt to
validate the 'detached signature' file `Release.gpg` which is a digital
signature for the `Release` file created using the signing key for the repo.
For this check to work, the script will need access to the __public__ key for
the repository.  It will attempt to find this key by looking in each of the
`*.gpg` keyring files in `/etc/apt/trusted.gpg.d/` and failing that it will
also look in the legacy keyring file `/etc/apt/trusted.gpg`.  If the public key
cannot be found (e.g.: due to permissions on the keyring files), the signature
validation phase will be silently skipped.



Title: Verification

# Verification

### Verify Release Packages

It is essential that you verify the integrity of the downloaded 
files using the PGP signatures. Please read 
[Verifying Apache HTTP Server Releases](http://httpd.apache.org/dev/verification.html) 
for more information on why you should verify our releases.

The PGP signatures can be verified using PGP or GPG. First download 
the [KEYS](http://www.apache.org/dist/olingo/KEYS) file as well 
as the .asc signature files for the relevant release packages. Make sure 
you get these files from the 
[main distribution directory](http://www.apache.org/dist/olingo/), 
rather than from a mirror. Then verify the signatures using

```sh
    % pgpk -a KEYS
    % pgpv <file>.asc
```

or

```sh
    % pgp -ka KEYS
    % pgp <file>.asc
```

or

```sh
    % gpg --import KEYS
    % gpg --verify <file>.asc
```
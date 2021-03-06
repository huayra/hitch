# Configuring Hitch

Hitch can be configured either from command line arguments or from a
configuration file on disk.

You can extract the usage description by invoking Hitch with the "--help"
argument. An example configuration file is included in the distribution.

In general Hitch is a protocol agnostic proxy and does not need much configuration.

List of configuration items to consider:

  - PEM files with key and certificate.
  - Listening addresses and ports. Note the semi-odd square brackets for IPv4 addresses.
  - Which backend servers to proxy towards, and if PROXY protocol should be used.
  - Number of workers, usually 1. For larger setups, use one worker per core.

If you need to support legacy clients, you can consider:

  - Enable SSLv3 with "--ssl" (despite RFC7568.)
  - Use weaker ciphers.

## Specifying ciphers

The recommended default is:

    "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH"

If you need to support legacy clients, consider the "HIGH" cipher group.

Normally you do not have to change this.


## Run environment

If you're handling a large number of connections, you'll probably want to raise
`ulimit -n` before running Hitch.

If you are listening to ports under 1024 (443 comes to mind), you need
to start Hitch as root. In those cases you *must* use --user/-u to set
a non-privileged user `hitch` can setuid() to.


## Preparing PEM files

PEM files should contain the key file, the certificate from the CA and any
intermediate CAs needed.

    $ cat example.com.key intermediate.pem example.com.crt > example.com.pem

If you want to use Diffie-Hellman based ciphers for Perfect Forward Secrecy
(PFS), you need to add some parameters for that as well:

    $ openssl dhparam -rand - 2048 >> example.com.pem

Hitch will complain and disable DH unless these parameters are available.

## OCSP stapling

Hitch has support for loading and stapling of OCSP responses. If
configured, Hitch will include a stapled OCSP response as part of the
handshake when it recieves a status request from a client.

Retrieving an OCSP response suitable for use with Hitch can be done
using the following `openssl` command:

    $ openssl ocsp \
        -url https://ocsp.example.com \
        -header Host ocsp.example.com \
        -no_nonce \
        -resp_text \
        -issuer issuer.pem \
        -cert mycert.pem \
        -respout ocspresp.der

This will produce a DER-encoded OCSP response which can then be loaded
by Hitch.

The URL of the OCSP responder can be retrieved via

	$ openssl x509 -ocsp_uri -in mycert.pem -noout

the `-issuer` argument needs to point to the OCSP issuer
certificate. Typically this is the same certificate as the
intermediate that signed the server certificate.

To configure Hitch to use the OCSP staple, use the following
incantation when specifying the `pem-file` setting in your Hitch
configuration file:

    pem-file = {
        cert = "mycert.pem"
        ocsp-resp-file = "mycert-ocsp.der"
    }


## Uninterrupted configuration reload

Issuing a SIGHUP signal to the main Hitch process will initiate a
reload of Hitch's configuration file.

Adding, updating and removing PEM files (``pem-file``) and frontend
listen endpoints (``frontend``) is currently supported.

Hitch will load the new configuration in its main process, and spawn a
new set of child processes with the new configuration in place if
successful.

The previous set of child processes will finish their handling of any
live connections, and exit after they are done.

If the new configuration fails to load, an error message will be
written to syslog. Operation will continue without interruption with
the current set of worker processes.

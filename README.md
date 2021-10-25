hosted-ce-tools
===============
Tools for maintaining an OSG Hosted CE

This repo contains scripts, services, timers, etc. that are useful for managing an OSG Hosted CE.


update-remote-wn-client
-----------------------

This script does the following for a single remote host:

- download and extract a worker node tarball
- set up the worker node paths to work on the remote host
- download the latest CA certs and CRLs into the worker node installation
- rsync the above to the remote host

Run it as the user that will be doing bosco submission.
An SSH key already needs to be set up on the remote host.
Run `update-remote-wn-client --help` for a list of arguments and defaults.


update-all-remote-wn-clients
----------------------------

This service periodically runs `update-remote-wn-client` for a list of endpoints.
The endpoints are specified as sections in `/etc/endpoints.ini`.
An example endpoint is provided in the default config file.

This can be run as a script or as a systemd service.
To run it as a service:

```
# systemctl enable --now update-all-remote-wn-clients
```

Logs will be in `/var/log/update-remote-wn-client`.
Each endpoint will have its own logfile.


cvmfsexec-osg-wrapper
---------------------

Uses CVMFSEXEC to mount some CVMFS repos and runs the executable given on the
command line.  Meant to be run on the exec node as a job wrapper around
the pilot itself; copy it to a location accessible from the pilot job and set
it as `blah_job_wrapper` in `etc/blah.config`, such as

    blah_job_wrapper="$HOME/cvmfsexec-osg-wrapper -safe"

`-safe` runs the wrapped job even if mounting CVMFS failed.  The reason for
the failure will be stored in the environment variable `CVMFSEXEC_WRAPPER_FAILURE`.

As of 2021-10-25, downloads CVMFSEXEC on the fly but we plan to change that
to use a tarball generated by `make-cvmfs-tarball` (see below) instead.

make-cvmfsexec-tarball
----------------------

Downloads CVMFSEXEC from Git, runs `makedist osg`, and tars up the result.
Not used yet (as of 2021-10-25); the plan is to run it on the hosted CE, copy
the result to the remote site, and use it in `cvmfsexec-osg-wrapper` (see
above).


Known issue(s)
--------------

- `update-all-remote-wn-clients` separately downloads the tarball, CAs, and CRLs
  for each endpoint in the configuration


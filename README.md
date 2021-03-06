## rhel-repo-sync

Is an `ansible` playbook allowing you to host multiple Red Hat version
repositories all on one (or more) servers. By retrieving and refactoring the SSL
certificates and repository information from a given subscribed server(s) thus
allowing the host or hosts to be able to syncronise with Red Hat CDN as if it
was a subscribed host.

The example below uses two existing Red Hat servers, one RHEL6 and one RHEL7.
Both machines have a valid subscription for the repositories we want to host.

Simply configure in the `vars` the variable `servers` describing the `FQDN` and
what Red Hat `release` the server is, and finally a list of the `repos` you are
wanting to to syncronise with Red Hat CDN.

```
    servers:
      rhel7.internal.lan:
        release: 7Server
        repos:
          - rhel-7-server-rpms
          - rhel-7-server-optional-rpms
          - rhel-7-server-extras-rpms
          - rhel-7-server-supplementary-rpms
          - rhel-ha-for-rhel-7-server-rpms
          - rhel-rs-for-rhel-7-server-rpms
          - rhel-server-rhscl-7-rpms
      rhel6.internal.lan:
        release: 6Server
        repos:
          - rhel-6-server-extras-rpms
          - rhel-6-server-optional-rpms
          - rhel-6-server-rpms
          - rhel-6-server-supplementary-rpms
          - rhel-server-rhscl-6-rpms
```
You can control the destination location of the repositories base directory by
setting the `repo_base` variable. By default it is set to `/var/www/html/repos`

A sample redacted structure post execution is:
```
/var/www/html/repos/
├── 6Server
│   ├── rhel-6-server-extras-rpms
│   │   ├── Packages
│   │   └── repodata
│   ├── rhel-6-server-optional-rpms
│   │   ├── Packages
│   │   └── repodata
│   ├── rhel-6-server-rpms
│   │   ├── Packages
│   │   └── repodata
│   ├── rhel-6-server-supplementary-rpms
│   │   ├── Packages
│   │   └── repodata
│   └── rhel-server-rhscl-6-rpms
│       ├── Packages
│       └── repodata
└── 7Server
    ├── rhel-7-server-extras-rpms
    │   ├── Packages
    │   └── repodata
    ├── rhel-7-server-optional-rpms
    │   ├── Packages
    │   └── repodata
    ├── rhel-7-server-rpms
    │   ├── Packages
    │   └── repodata
    ├── rhel-7-server-supplementary-rpms
    │   ├── Packages
    │   └── repodata
    ├── rhel-ha-for-rhel-7-server-rpms
    │   ├── Packages
    │   └── repodata
    ├── rhel-rs-for-rhel-7-server-rpms
    │   ├── Packages
    │   └── repodata
    └── rhel-server-rhscl-7-rpms
        ├── Packages
        └── repodata
```
All configurations used by the `reposync` command are generated and placed in a
directory which is configurable by editing the `config_path` variable. The
default is set to `config_path: /etc/reposync` and contains a custom `yum.conf`
file, the refactored `.repo` files, and the SSL certificates. An example:
```
/etc/reposync/
├── certs
│   ├── 6Server
│   │   ├── 6447223819138729323-key.pem
│   │   ├── 6447223819138729323.pem
│   │   └── redhat-uep.pem
│   └── 7Server
│       ├── 0990874270948909603-key.pem
│       ├── 0990874270948909603.pem
│       ├── 2576679062146528490-key.pem
│       ├── 2576679062146528490.pem
│       ├── 8772713078987633877-key.pem
│       ├── 8772713078987633877.pem
│       └── redhat-uep.pem
├── repos.d
│   ├── rhel-6-server-extras-rpms.repo
│   ├── rhel-6-server-optional-rpms.repo
│   ├── rhel-6-server-rpms.repo
│   ├── rhel-6-server-supplementary-rpms.repo
│   ├── rhel-7-server-extras-rpms.repo
│   ├── rhel-7-server-optional-rpms.repo
│   ├── rhel-7-server-rpms.repo
│   ├── rhel-7-server-supplementary-rpms.repo
│   ├── rhel-ha-for-rhel-7-server-rpms.repo
│   ├── rhel-rs-for-rhel-7-server-rpms.repo
│   ├── rhel-server-rhscl-6-rpms.repo
│   └── rhel-server-rhscl-7-rpms.repo
└── yum.conf
```

Each `.repo` file is generated by extracting the INI section and refactoring it
from the original `redhat.repo` file present on the subscribed server. Changes
made are that we point the SSL certificates and key to the new path. Hardcode
the `releasever` and set the `basearch` to `x86_64` (May add other architecture
options if requested).

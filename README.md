Minimal Ubuntu images for Docker
================================

Ever wondered why Ubuntu for Docker comes with *systemd* and tools for filesystem management?
Yeah, [me](https://twitter.com/murmosh) too.
These are container images without that fuzz.

* blitznote/debootstrap-amd64:16.10 [![](https://images.microbadger.com/badges/image/blitznote/debootstrap-amd64:16.10.svg)](http://microbadger.com/images/blitznote/debootstrap-amd64 "Get your own image badge on microbadger.com")
* blitznote/debootstrap-amd64:16.04 [![](https://images.microbadger.com/badges/image/blitznote/debootstrap-amd64:16.04.svg)](http://microbadger.com/images/blitznote/debootstrap-amd64 "Get your own image badge on microbadger.com")
* blitznote/debootstrap-amd64:15.10 [![](https://images.microbadger.com/badges/image/blitznote/debootstrap-amd64:15.10.svg)](http://microbadger.com/images/blitznote/debootstrap-amd64 "Get your own image badge on microbadger.com") [EOL]

If in doubt use `16.04`.

![size comparison: Ubuntu for Docker 120MB, ubuntu-debootstrap 87MB, blitznote/debootstrap-amd64 55MB](https://rawgit.com/Blitznote/docker-ubuntu-debootstrap/master/ubuntu-for-Docker-sizes.svg)

[![Docker image](https://img.shields.io/badge/Docker-blitznote%2Fdebootstrap-blue.svg)](https://hub.docker.com/r/blitznote/debootstrap-amd64/) ← https://github.com/Blitznote/docker-ubuntu-debootstrap

Features
--------

* **small**:
  * 63% the size of ubuntu-debootstrap (:16.04@898cb62b7368)
  * 45% the size of ubuntu (:16.04@44776f55294a)
* comes with *apt-transport-https*
* and latest *curl*
* a bootstrap *[ca-certificates.crt](https://github.com/wmark/docker-curl/blob/master/ca-certificates.crt)*
* latest *signify* for Linux from [Blitznote/signify](https://github.com/Blitznote/signify)
* bzip2, jq, plzip, runit (for its *chpst*), unzip
* with locale ISO.UTF-8 as default

Usage
-----

This is meant as drop-in replacement for ```FROM ubuntu``` and ```FROM ubuntu-debootstrap```.

You can use *curl* right away or start with ```apt-get -q update``` as usual.
HTTPS support is already included in *apt*!

Find examples here:

* [Mark's Docker images](https://hub.docker.com/u/wmark/) – [Github](https://github.com/search?q=user%3Awmark+docker-)

### Recommendations

Use lightweight *chpst* (31 kB) instead of *gosu* (2635 kB):

```diff
- gosu myuser syncthing "$@"
+ chpst -u myuser -- syncthing "$@"
- gosu nobody:root bash -c 'whoami && id'
+ chpst -u nobody:root -- bash -c 'whoami && id'
```

To account for differences between *gpg v1* and *gpg v2*
I've created a script for fetching keys from keyservers:

```bash
/usr/bin/get-gpg-key 0xcbcb082a1bb943db 0xa6a19b38d3d831ef \
| apt-key add
```

### Troubleshooting

These images come with latest versions of **libc** and **Perl**, no matter which
Ubuntu version you have downloaded.
Although this decision benefits security, you can run into packages which won't install due to dependencies.
In almost all cases it's either related to *Perl* or broken in vanilla Ubuntu, too.

To downgrade *Perl* to use packages from Ubuntu only:

```bash
. /etc/os-release

# 16.04 - xenial
# 16.10 - yakkety
apt-get -t ${UBUNTU_CODENAME} --allow-downgrades -y install perl-base=5.22*
```

If you intend to use this Docker image for building packages, you will need to uncompress
the shipped version files first:

```bash
find /var/lib/dpkg/info -type f -name '*.lz' \
| xargs --max-args=1 --max-procs=$(nproc) --no-run-if-empty \
  plzip --keep -d
```

Regenerate the Images
---------------------

1. Use the packages sources from `/etc/apt/sources.list`.
2. Install the packages listed in `build.manifest` using **apt**.
3. Remove any excess that cannot be used from within an container.

I have published a script which automates that.
[You can find it on Github, as answer to question/issue 2](https://github.com/Blitznote/docker-ubuntu-debootstrap/issues/2#issuecomment-256456602).

Caveats
-------

* Images for architecture *amd64*/x86_64 require instruction set **SSE 4**, which had been introduced in 2007.
  If you don't have a reasonably recent CPU you will eventually run into the `illegal instruction` error.
* You need Linux 3.10.0 or later. Version 3.18.0 or later is strongly recommended.
* Kernel support for **Seccomp** is strongly recommended.
  `zgrep SECCOMP /proc/config.gz`

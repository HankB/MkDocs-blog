# Sanoid issue #813 testing

<https://github.com/jimsalterjrs/sanoid/issues/813>

## 2023-04-30 lets get started

Build instructions at <https://github.com/jimsalterjrs/sanoid/blob/master/INSTALL.md#debianubuntu>

```text
apt install debhelper libcapture-tiny-perl libconfig-inifiles-perl pv lzop mbuffer build-essential git
git clone https://github.com/jimsalterjrs/sanoid.git
cd sanoid
# checkout latest stable release or stay on master for bleeding edge stuff (but expect bugs!)
git checkout $(git tag | grep "^v" | tail -n 1)
ln -s packages/debian .
dpkg-buildpackage -uc -us
sudo apt install ../sanoid_*_all.deb
```

Staying on the bleeding edge. After installing the dependencies, the package built. purging and installing left the previous `/etc/sanoid/sanoid.conf`. No change to version number so it will be necessary to purge/install for each test host. Changing VERISON to `2.1.0a` did not change this. Did need to re-enable the timer.

```text
sudo systemctl enable --now sanoid.timer
```

### New features

* `--insecure-direct-connection`
* `--preserve-compression`
* `--no-forced-receive`


### Monitoring

Hosts

* `mesquite` FreeBSD, `sanoid` 2.1.0.
* `cm4iob` Debian 12 (Bookworm) `sanoid` 2.1.0 ->  `55c5e0e`
* `piserver` Debian 12 (Bookworm) `sanoid` 2.1.0
* `kweli` Debian 12 (bookworm) `sanoid` 

* Twice hourly backup `cm4iob` -> `mesquite` (BSD) at 45,25.
* Add hourly backup `cm4iob` -> `piserver` (Debian/ARM64)

Results

1. Upgrade `cm4iob` to `55c5e0e`
1. First hourly `cm4iob` -> `mesquite`
1. Seed (first syncoid) `cm4iob` -> `piserver` 
1. automate `cm4iob` -> `piserver` hourly at 32
1. first hourly `cm4iob` -> `piserver` failed: `Target tank/backup/cm4iob exists but has no snapshots matching with e4d25c8051695501/backup/backup` Manual (with `-f`) was successful. Need to check in more detail next time a host is seeded.
1. 2023-05-01
1. `apt` suggests `sanoid` upgrade from repo. `apt-mark hold sanoid`

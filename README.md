# t3nk3y/bees

Dockerized Best-Effort Extent-Same btrfs deduplication agent.
https://github.com/Zygo/bees

This image is based on Ubuntu Linux with bees agent from testing repositories.

We've cloned the github repo: [deatheibon/bees](https://github.com/deatheibon/bees-docker), most of the credit for the dockerization should go to them.

### Source Code
Source code is available on GitHub here: https://github.com/t3nk3y/bees-docker

## Usage

Bees only works with mounted BTRFS root subvolumes.  It does not work on any other subvolumes, so you need to mount your root subvolume of the filesystem to your native environment.  If you have more than one subvolume, the bees documentation suggests using an alternative mount point that isn't used in other places. Example:

```Shell
mount /dev/"device" /mnt/btrfs_roots/mount_name -o subvol=/
mount /dev/"device" /mnt/btrfs_roots/another_mount_name -o subvol=/
```

Be sure to adjust the docker-compose volumes section.  There should be one servive for each volume you wish to dedupe.  Change the service name and the mount volume path(the last line in each service in the example.)

## configuration
You can adjust the BTRFS confiuration by settings the environment variables:

### HASH_TABLE
This sets the locaiton of the hash table(relative to the docker environment).  The examples will set it to a hidden folder in the BTRFS root mount for each device you mount:
```
HASH_TABLE=/mnt/.beeshome/beeshash.dat
```

### HASH_TABLE_SIZE
This sets the hash table size.  The [bees documentation](https://github.com/Zygo/bees/blob/master/docs/config.md) suggests 128K, but you can read the [bees documentation](https://github.com/Zygo/bees/blob/master/docs/config.md) to decide on what would be best for your situation.
```
HASH_TABLE_SIZE=128K
```

### OPTIONS
This sets any options that must be configured on the bees commandline.  The examples will set -a to workaround a known problem with btrfs send and -v 6 to reduce the excessive log verbosity from the default of debug mode to warnings or errors only.

For further options, see the [bees documentation](https://github.com/Zygo/bees/blob/master/docs/options.md)
```
OPTIONS=-a -v 6
```
## docker run example

### daemonized
```Shell
docker run -d --privileged -e HASH_TABLE=/mnt/.beeshome/beeshash.dat -e HASH_TABLE_SIZE=128K -e OPTIONS="-a -v 6" -v /etc/localtime:/etc/localtime:ro -v /etc/timezone:/etc/timezone:ro -v /path/to/btrfs/mount:/mnt t3nk3y/bees
```

### interactive
```Shell
docker run -ti --privileged -e HASH_TABLE=/mnt/.beeshome/beeshash.dat -e HASH_TABLE_SIZE=128K -e OPTIONS="-a -v 6" -v /etc/localtime:/etc/localtime:ro -v /etc/timezone:/etc/timezone:ro -v /path/to/btrfs/mount:/mnt t3nk3y/bees
```

## docker-compose example
```YAML
version: '3.8'
services:
  mount_name:
    image: t3nk3y/bees
    privileged: true
    restart: unless-stopped
    hostname: beesd
    environment:
      - HASH_TABLE=/mnt/.beeshome/beeshash.dat
      - HASH_TABLE_SIZE=128K
      - OPTIONS=-a -v 6
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - /path/to/btrfs/mount:/mnt

  another_mount_name:
    image: t3nk3y/bees
    privileged: true
    restart: unless-stopped
    hostname: beesd
    environment:
      - HASH_TABLE=/mnt/.beeshome/beeshash.dat
      - HASH_TABLE_SIZE=128K
      - OPTIONS=-a -v 6
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - /path/to/btrfs/another_mount:/mnt
```

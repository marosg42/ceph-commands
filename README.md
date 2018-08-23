# Usefull Ceph commands

Hints and tips for Ceph

## Terminology

- RADOSGW - objects storage, bucket based REST gateways, compatible with S3 and swift
- RBD - block storage, block device with Linux kernel client and QEMU/KVM driver
- CEPH FS - distributed filesystem
- RADOS - reliable autonomous object store comprised of inteligent self-healing storage nodes

![Ceph architecture](stack.png)

## Health and overview commands

`ceph -s` - see overall status of the cluster, number of OSDs, monitors etc

`ceph health detail` - good for checking which OSDs have problems

`ceph -w` - continuous display of what is happening inside cluster

## monitors

## OSDs

- `ceph daemon osd.0 config show` - show active config

- `ceph daemon osd.0 config diff` - show difference between defaults and active config

- `ceph tell osd.0 injectargs --debug-osd 20 --debug-ms 1` - runtime changes to an daemon

- `ceph tell osd.* injectargs '--rbd_cache_max_dirty_age = 1'` - runtime change in all OSDs

## Pools

`ceph osd pool ls`

`ceph osd pool ls detail` - much better than previous one

`rados -p <poolname> ls` - pool info from rados point of view

`rbd -p glance ls` - list objects in a given pool

`rbd -p glance ls --long` - # list objects in a given pool, more details

## Crushmap

`ceph osd crush reweight <name> <number>` - reweight OSD

## PGs

`ceph pg dump` - dump info about **all** PGs onto screen, huge output

`ceph pg dump pgs_brief` - less clutter

`ceph pg dump osds` - more targetted, less clutter

`ceph pg dump pools` - more targetted, less clutter

`ceph pg ls stale/active/etc` - display PGs by state

`ceph pg {poolnum}.{pg-id} query` - very detailed info about single PG

`ceph pg map <pg>`

## rados

`rados df` - show pool utilization stats, sizes etc, execute on monitor
`rados lspools` - show pool utilization stats, sizes etc, execute on monitor

## rgw
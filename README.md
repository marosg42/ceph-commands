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

`ceph df / ceph df detail` - very nice overview, pools, sizes, objects etc

`ceph auth ls` - list authorized clients

## dynamic configuration

`ceph -n osd.123 --show-config` - very misleading, shows default config, not actual parameters

`ceph daemon (or --admin-daemon) /var/run/ceph/ceph-osd.1.asok config show` - shows actual configuration

### admin socket

`ceph daemon {daemon-name} or ceph daemon {path-to-socket-file}`

`ceph daemon osd.0 foo or ceph daemon /var/run/ceph/ceph-osd.0.asok foo`

`ceph daemon {daemon-name} help` - To view the available admin socket commands

`ceph daemon /run/ceph/ceph-mon.....asok mon_status` - mon status using admin socket

`ceph daemon $(ls /var/run/ceph/*asok) help`

`ceph tell osd.\* injectargs '--osd_backfill_full_ratio 0.92` - change parameter dynamically

`ceph tell osd.\* injectargs '--osd_backfill_scan_min 16 --osd_backfill_scan_max 32'` - change multiple parameters

`ceph tell mon.* injectargs '--mon-data-avail-warn 20'` - change monitor paramaters dynamically

## OSDs

`ceph osd stat`

`ceph osd tree`

`ceph osd lspools`

`ceph osd df` - usage of all OSDs

`ceph osd df|sort -n -k 7|uniq` - show usage on OSDs and sort them by %full

`ceph daemon osd.0 config show` - show active config

`ceph daemon osd.0 config diff` - show difference between defaults and active config

`ceph tell osd.0 injectargs --debug-osd 20 --debug-ms 1` - runtime changes to an daemon

`ceph tell osd.* injectargs '--rbd_cache_max_dirty_age = 1'` - runtime change in all OSDs

`systemctl start ceph-osd@{osd-num}` - start OSD in systemd system

`start ceph-osd id={osd-num}` - start OSD in trusty

`ceph osd repair <osd>`

`ceph-disk list` - run on OSD, show all info about OSD disks

`ceph osd add-<flag> <osd-id>` - set per-OSD flags

- noup: OSD is not allowed to start
- nodown: failure reports for this OSD will be ignored
- noin: if this OSD was previously marked out automatically after a failure, it will not be marked in when it starts
- noout: if this OSD is down it will not automatically be marked out after the configured interval
  
`ceph osd rm-<flag> <osd-id>` - unset per-OSD flags

`ceph-disk zap /dev/...` - run on OSD, format disk for OSD use

`ceph osd map <poolname> objectid` - (objectid is for example glance image id ) - shows mapping object to PG

`ceph --admin-daemon /var/run/ceph/ceph-osd.2.asok perf dump` - performance counters

`ceph osd blocked-by` - shows a list of OSDs which are blocking other OSD, also a number of blocked OSD per each blocker

`ceph daemon osd.x dump_ops_in_flight` - shows active operations on a given OSD. Command executed on the OSD node

## Pools

`ceph osd pool ls`

`ceph osd pool ls detail` - much better than previous one

`rados -p <poolname> ls` - pool info from rados point of view

`rbd -p glance ls` - list objects in a given pool

`rbd -p glance ls --long` - # list objects in a given pool, more details

`ceph osd pool create rbd1 <pgnum> <pgpnum> replicated` - create replicated pool

`ceph osd pool set {pool-name} pg_num {pg_num}` - set pgnum for existing pool, shoul dbe immediatelly followed by `ceph osd pool set {pool-name} pgp_num {pgp_num}`

`ceph osd pool delete int.rgw.buckets.data int.rgw.buckets.data --yes-i-really-really-mean-it` - remove pool

`ceph osd pool get {pool-name} {key}` - get pool values

`ceph osd pool set {pool-name} {key} {value}` - set pool value

`ceph osd pool stats` - show what is going on per pool

`ceph pg ls-by-pool <pool_name>` - list of PGs in a given pool

`ceph osd pool application enable <poolname> <app> {--yes-i-really-mean-it}` - To enable a client application to conduct I/O operations on a pool

- Where app is:
  - cephfs for the Ceph Filesystem.
  - rbd for the Ceph Block Device
  - rgw for the Ceph Object Gateway

## Crushmap

`ceph osd crush reweight <name> <number>` - reweight OSD

`ceph osd crush tree`

`ceph osd crush remove nodeX`

## PGs

`ceph pg dump` - dump info about **all** PGs onto screen, huge output

`ceph pg dump pgs_brief` - less clutter

`ceph pg dump osds` - more targetted, less clutter

`ceph pg dump pools` - more targetted, less clutter

`ceph pg ls-by-pool <pool_name>` - list of PGs in a given pool

`ceph pg ls stale/active/etc` - display PGs by state

`ceph pg dump_stuck inactive|unclean|stale`
  
- Inactive PGs cannot process reads or writes because they are waiting for an OSD with the most up-to-date data to come up and in.

- Unclean PGs contain objects that are not replicated the desired number of times. They should be recovering.

- Stale PGs are in an unknown state - the OSDs that host them have not reported to the monitor cluster in a while (configured by mon_osd_report_timeout).

`ceph pg {poolnum}.{pg-id} query` - very detailed info about single PG

`ceph pg map <pg>` - Ceph will return the placement group map, the placement group, and the OSD status

- returns osdmap eNNN pg {pg-num} -> up [0,1,2] acting [0,1,2]  (NNN is epoch) If the Up Set and Acting Set do not match, this may be an indicator that the cluster rebalancing itself or of a potential problem with the cluster

`ceph pg scrub {pg-id}` - scrub individual PG

`ceph pg {pg-id} mark_unfound_lost revert|delete` - mark the unfound objects as lost

`ceph pg stat` - The result should tell the placement group map version (vNNNNNN), the total number of placement groups (x), and how many placement groups are in a particular state such as active+clean (y)

`ceph tell <pgid> query` - state of specific problematic PG

`ceph pg ls-by-pool <pool_name>` - list of PGs in a given pool

## rados

`rados df` - show pool utilization stats, sizes etc, execute on monitor
`rados lspools` - show pool utilization stats, sizes etc, execute on monitor

# block devices

`rbd du <-p poolname>` - block devices are thin provisioned, this shows actual usage

`rbd create image2 --size 16G --pool <poolname>` - create block device in a given pool

`rbd ls <--pool x>` - list images in a pool

`rbd --image <imagename> -p <poolname> info` - display detailed info about a volume

## rgw

`radosgw-admin bucket stats` - list all buckets in radosgw

`radosgw-admin --id radosgw.gateway -k /etc/ceph/keyring.rados.gateway bucket stats 2>&1` - list of buckets and their properties

## Playbooks

[Some playbooks in a separate file](playbooks.md)

## Documentation

[Ceph command reference](http://docs.ceph.com/docs/master/rados/operations/control/#)

[List of all health warnings](http://docs.ceph.com/docs/master/rados/operations/health-checks/)

[Placement groups description](http://docs.ceph.com/docs/master/rados/operations/placement-groups/)
[Resolve a Volume is Busy Error on Cinder With Ceph Block Storage](https://mcwhirter.com.au/craige/blog/2015/Cinder_Cannot_Delete_Volume_-_volume_is_busy_With_Ceph_Block_Storage/)

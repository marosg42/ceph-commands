# Playbooks

## Define new osd to ceph

```ENG
ceph osd create <UUID> <id>
ceph osd crush add <id> 1.09099 host=<hostname>
ceph auth add osd.<id> osd 'allow *' mon 'allow profile osd' -i keys
```

## Remove OSD

```ENG
ceph osd out $o
ceph osd crush remove $o
ceph auth del $o
ceph osd rm $o
```

## Unlock after crashed client

```ENG
$ sudo ceph osd lspools
1 default.rgw.buckets,2 default.rgw,3 default.rgw.root,4 default.rgw.control,5 default.rgw.gc,6 default.rgw.buckets.index,7 default.rgw.buckets.extra,8 default.log,9 default.intent-log,10 default.usage,11 default.users,12 default.users.email,13 default.users.swift,14 default.users.uid,15 .rgw.root,16 default.rgw.meta,17 default.rgw.log,18 glance,19 cinder-ceph,

$ sudo rbd lock list cinder-ceph/volume-9707c7e8-f67d-4028-ba72-5cf6009fbf66
There is 1 exclusive lock on this image.
Locker       ID                   Address
client.84541 auto 140088344329632 192.168.100.86:0/2248567790

$ sudo rbd lock remove cinder-ceph/volume-9707c7e8-f67d-4028-ba72-5cf6009fbf66 ""auto 140088344329632"" client.84541

$ sudo rbd lock list cinder-ceph/volume-9707c7e8-f67d-4028-ba72-5cf6009fbf66
```

## Playing with RBD

```ENG
modprobe rbd

copy /etc/ceph/ceph.client...keyring from monitor

rbd create image-for-test --size 4096 --pool <poolname>

in a case above is not enough, here are all needed parms
    rbd create image-for-test --size 4096 --pool <poolname> --keyring <path-to-keyring> --id <identity>

rbd ls --pool <poolname>
in a case above is not enough, here are all needed parms
    rbd ls -k <path-to-keyring> --name <identity?> --pool <poolname>

rbd feature disable image-for-test exclusive-lock object-map fast-diff deep-flatten

rbd map image-for-test --pool <poolname>
```
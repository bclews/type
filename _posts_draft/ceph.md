---
layout: post
title: Ceph
---

# Server Details

Ubuntu 19.10.

## clews-01
- private: 10.19.96.3
- public:  104.156.232.110 

## clews-02
- private: 10.19.96.4
- public:  139.180.172.162 

## clews-03
- private: 10.19.96.5
- public:  149.28.172.62 

# Configuring a private network on Vultr
See:

- [https://www.vultr.com/docs/configuring-private-network](https://www.vultr.com/docs/configuring-private-network).
- [https://www.keycdn.com/support/what-is-cidr](What Is CIDR (Classless Inter-Domain Routing)?)
- [https://www.vultr.com/resources/subnet-calculator/?ip_address=10.19.96.3%2F30&mask_bits=11](IPv4 Subnet Calculator)

The networking details can be found under `Settings > Public Network > View our networking configuration tips and examples`

Test it out by pinging one of the other private IP addresses.

# Creating a Docker Swarm

## Install docker on each machine
```bash
apt  install docker.io
```

On the first node:
```bash
docker swarm init --advertise-addr 10.19.96.3
docker swarm join-token manager
```

On the remaining nodes:
```bash
docker swarm join --token SWMTKN-1-4la0gyzfyxfksezu84d8tjq7i07n1u8m1e4tzme7bh3hfgdin6-8dwv5ezq9o9ajd5mdz1uztnyb 10.19.96.3:2377
```

To see the swarm:
```bash
docker node ls
```

Which should show:
```bash
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
i9euclq1v0jpy5mwlxvmvypge *   clews-01            Ready               Active              Leader              19.03.2
8ve1uiivn1mnojcwxhig2gswz     clews-02            Ready               Active              Reachable           19.03.2
tkhw9255px2xkc2iwbe9lmfjj     clews-03            Ready               Active              Reachable           19.03.2
```

# Funky Penguin - Nodes

## Install ceph on Ubuntu
Ubuntu doesn't come with a ceph client. Install now to avoid [issues later](https://lists.ceph.io/hyperkitty/list/ceph-users@ceph.io/thread/NHERSWSI3NLUBWESKZK4TDR6KGSHO5VD/):
```
sudo apt install ceph-common
```

# Shared Storage (Ceph)

This was useful, [ceph-docker](https://hub.docker.com/r/ceph/daemon/)
``` bash
ceph --version
ceph version 14.2.2 (4f8fa0a0024755aae7d95567c63f11d6862d55be) nautilus (stable)
```

So we want to be running the nautilus:
```
docker pull ceph/daemon:latest-nautilus
```

## Setup Monitors

```
docker run -d --net=host \
--restart always \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-e MON_IP=10.19.96.3 \
-e CEPH_PUBLIC_NETWORK=10.19.96.3/20 \
--name="ceph-mon" \
ceph/daemon:latest-nautilus mon
```

I tried this and got errors
```
root@clews-01:~# docker logs -f ceph-mon
importing contents of /var/lib/ceph/bootstrap-osd/ceph.keyring into /etc/ceph/ceph.mon.keyring
importing contents of /var/lib/ceph/bootstrap-mds/ceph.keyring into /etc/ceph/ceph.mon.keyring
importing contents of /var/lib/ceph/bootstrap-rgw/ceph.keyring into /etc/ceph/ceph.mon.keyring
importing contents of /var/lib/ceph/bootstrap-rbd/ceph.keyring into /etc/ceph/ceph.mon.keyring
importing contents of /etc/ceph/ceph.client.admin.keyring into /etc/ceph/ceph.mon.keyring
2019-10-30 02:15:34.310945 7f46f9dd3000 -1 stat(/var/lib/ceph/mon/ceph-clews-01) (13) Permission denied
2019-10-30 02:15:34.311858 7f46f9dd3000 -1 error opening '/var/lib/ceph/mon/ceph-clews-01': (13) Permission denied
```

After googling, I found this link: [System IDs conflicts](https://www.objectif-libre.com/en/blog/2019/01/31/migrating-to-containerized-ceph-on-ubuntu-18-04-a-feedback/):
```
sed -i s/64045/167/g /etc/group /etc/passwd
chown 167:167 /etc/ceph
chown -R 167:167 /var/lib/ceph
chown 167:167 /var/log/ceph
reboot
```

I then re-ran:
```
docker run -d --net=host \
--restart always \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-e MON_IP=10.19.96.3 \
-e CEPH_PUBLIC_NETWORK=10.19.96.3/20 \
--name="ceph-mon" \
ceph/daemon:latest-nautilus mon
```

Aaaaand.... dammit!!!
```
2018-06-19 13:44:01.542990 7ff75b024700  1 mon.vm02@1(peon).log v57928205 unable to write to '/var/log/ceph/ceph.log' for channel 'cluster': (2) No such file or directory
```

Based on [this github issue](https://github.com/ceph/ceph-container/issues/1094) I added the following to `/etc/ceph/ceph.conf`:
```
[mon]
mon_cluster_log_file = /dev/null
```

Aaaaand.... it seems to have worked...

For the other two servers I ran:
```
sed -i s/64045/167/g /etc/group /etc/passwd
chown 167:167 /etc/ceph
chown -R 167:167 /var/lib/ceph
chown 167:167 /var/log/ceph
reboot
```

And then copy the generated data over to the other 2 nodes:
```
scp -r /etc/ceph/* root@139.180.172.162:/etc/ceph/
scp -r /etc/ceph/* root@149.28.172.62:/etc/ceph/
```

And start the monitor on those also using the same command again (customizing MON_IP as you go). 

For server two:
```
docker run -d --net=host \
--restart always \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-e MON_IP=10.19.96.4 \
-e CEPH_PUBLIC_NETWORK=10.19.96.3/20 \
--name="ceph-mon" ceph/daemon:latest-nautilus mon
```

For server three:
```
docker run -d --net=host \
--restart always \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-e MON_IP=10.19.96.5 \
-e CEPH_PUBLIC_NETWORK=10.19.96.3/20 \
--name="ceph-mon" ceph/daemon:latest-nautilus mon
```

To verify the monitors are working, we can:
```bash
docker exec -it ceph-mon bash
ceph status
```

In the output we should see:
```bash
  services:
    mon: 3 daemons, quorum clews-01,clews-02,clews-03 (age 0.296048s)
```

## Setup Managers
```
docker run -d --net=host \
--privileged=true \
--pid=host \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
--name="ceph-mgr" \
--restart=always \
ceph/daemon:latest-nautilus mgr
```

To verify the monitors are working, we can:
```bash
docker exec -it ceph-mgr bash
ceph status
```

In the output we should see:
```bash
  services:
    mon: 3 daemons, quorum clews-01,clews-02,clews-03 (age 4m)
    mgr: clews-01(active, since 95s), standbys: clews-02, clews-03
```

# Setup OSDs

Dump the auth credentials for the OSDs into the appropriate location on the base OS for each server.
```
ceph auth get client.bootstrap-osd -o \
/var/lib/ceph/bootstrap-osd/ceph.keyring
```

My ubuntu machine doesn't have `/dev/vdd` but it does have `/dev/vda1`. However, when I tried specifying an OSD_TYPE and OSD_DEVICE it kept restarting. So I removed them and let the daemon auto detect it (as per the docker hub instructions under *Deploy an OSD*):

```
docker run -d --net=host \
--privileged=true \
--pid=host \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /dev/:/dev/ \
--name="ceph-osd" \
--restart=always \
ceph/daemon:latest-nautilus osd
```

To verify the monitors are working, we can:
```bash
docker exec -it ceph-mgr bash
ceph status
```

In the output we should see:
```bash
  cluster:
    id:     0903d9cc-9cbc-40ff-979e-8f6b5f8e038a
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum clews-01,clews-02,clews-03 (age 6h)
    mgr: clews-01(active, since 6h), standbys: clews-02, clews-03
    osd: 3 osds: 3 up (since 4m), 3 in (since 4m)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   3.0 GiB used, 27 GiB / 30 GiB avail
    pgs:      
```

# Setup MDSs

```
docker run -d --net=host \
--name ceph-mds \
--restart always \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /etc/ceph:/etc/ceph \
-e CEPHFS_CREATE=1 \
ceph/daemon:latest-nautilus mds
```

```
docker run -d --net=host \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /etc/ceph:/etc/ceph \
-e CEPHFS_CREATE=1 \
ceph/daemon:latest-nautilus mds
```

# And follow the rest of the guide...

## Mount MDS volume
Quick note, the following didn't work. I had to replace the `$MYHOST` variable with the private IP address vultr assigned the machine.

```
MYHOST=`hostname -s`
echo -e "
# Mount cephfs volume \n
$MYHOST:6789:/      /var/data/      ceph      \
name=dockerswarm\
,secret=AQBp3ctd/aaDFxAAkxg4ISZoEl+LNmGyo3IEIw==\
,noatime,_netdev,context=system_u:object_r:svirt_sandbox_file_t:s0 \
0 2" >> /etc/fstab
mount -a
```

or
```
echo "$(hostname -s):6789:/      /var/data/      ceph      name=dockerswarm,secret=$(ceph-authtool /etc/ceph/keyring.dockerswarm -p -n client.dockerswarm),noatime,_netdev,context=system_u:object_r:svirt_sandbox_file_t:s0 0 2" >> /etc/fstab
```
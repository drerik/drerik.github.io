+++
Categories = ["Docker", "Blog"]
Description = ""
Tags = ["Docker", "notes", "blog"]
date = "2016-10-25T15:16:52+02:00"
menu = "posts"
title = "Debugging docker storage problems"

+++

# Docker storage full

```
d4a22e425a33: Download complete
a93bb7fbd2bc: Download complete
7468977abb55: Download complete
Pulling repository docker.io/enoniccloud/apache2
e450b1758d1a: Error pulling image (latest) from docker.io/enoniccloud/apache2, ApplyLayer exit status 1 stdout:  stderr: open /usr/lib/x86_64-linux-gnu/gconv/IBM1145.so: no space left on device
t on device2: Error downloading dependent layers
ERROR: Service 'apache2' failed to build: Error pulling image (latest) from docker.io/enoniccloud/apache2, ApplyLayer exit status 1 stdout:  stderr: open /usr/lib/x86_64-linux-gnu/gconv/IBM1145.so: no space left on device
```

checking disk usage
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        989M     0  989M   0% /dev
tmpfs          1003M     0 1003M   0% /dev/shm
tmpfs          1003M  1.5M 1002M   1% /run
tmpfs          1003M     0 1003M   0% /sys/fs/cgroup
/dev/vda9       7.4G  620M  6.4G   9% /
/dev/vda3       985M  492M  442M  53% /usr
/dev/vda1       128M   67M   61M  53% /boot
tmpfs          1003M     0 1003M   0% /media
tmpfs          1003M     0 1003M   0% /tmp
/dev/vda6       108M   60K   99M   1% /usr/share/oem
/dev/vdb1       9.8G  4.1G  5.2G  44% /var/lib/docker
```
hmm.. .still free space...

After google fu.. ( https://github.com/docker/docker/issues/10613 ) checking inodes... Ah!:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -i
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
devtmpfs        253044    330  252714    1% /dev
tmpfs           256700      1  256699    1% /dev/shm
tmpfs           256700    673  256027    1% /run
tmpfs           256700     14  256686    1% /sys/fs/cgroup
/dev/vda9      2019712    672 2019040    1% /
/dev/vda3       260096   6529  253567    3% /usr
/dev/vda1            0      0       0     - /boot
tmpfs           256700      1  256699    1% /media
tmpfs           256700      9  256691    1% /tmp
/dev/vda6        32768     13   32755    1% /usr/share/oem
/dev/vdb1       655360 652106    3254  100% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# df -h
Filesystem      Size  Used Avail Use% Mounted on
none             60G  6.6G   50G  12% /
tmpfs          1000M     0 1000M   0% /dev
tmpfs          1000M     0 1000M   0% /sys/fs/cgroup
osxfs           465G  319G  146G  69% /srv/home
shm              64M     0   64M   0% /dev/shm
/dev/vda2        60G  6.6G   50G  12% /etc/hosts
```

ok... Trying to delete dangling images:
```
core@esu-exp1-server-qhjozestjbvs ~ $ docker rmi $(docker images -q --filter "dangling=true")
Deleted: 116c7273831f195bfeefcf0b27d31007bac24b01a6231bceffc41dff6b2619e4
Deleted: 5bfd7f7c94a377763c2c79b0612edb75bee7004ad69d1167216df20e505c9d7f
Deleted: cdd82fbf60ba0e587ba0661da429675add51824f657ad14199cdbc8d261a4d51
Deleted: 132d5fbf54ee10b785ef9dde27ae59c0b7c692132cacb92f28a80d81bde63836
Deleted: a8897089c892dc71cb27eb137cfa96949c417a04a8345e5110593bb8a172ea83
Deleted: c64855f4e5ef19406976864670dbd12ba0962aeea47c6acd1bccc7d889cc094f
Error response from daemon: Conflict, cannot delete 86f47d28eef8 because the running container 1b84a98719f1 is using it, stop it and use -f to force
Error response from daemon: Conflict, cannot delete 86f47d28eef8 because the running container 1b84a98719f1 is using it, stop it and use -f to force
Error response from daemon: Conflict, cannot delete 86f47d28eef8 because the running container 1b84a98719f1 is using it, stop it and use -f to force
Error response from daemon: Conflict, cannot delete 86f47d28eef8 because the running container 1b84a98719f1 is using it, stop it and use -f to force
Error response from daemon: Conflict, cannot delete 86f47d28eef8 because the running container 1b84a98719f1 is using it, stop it and use -f to force
Error response from daemon: Conflict, cannot delete 86f47d28eef8 because the running container 1b84a98719f1 is using it, stop it and use -f to force
Deleted: 4493557196f9b662a0e091521c4b7a203a8c5333ce10d48fa8f4b369cf00be6d
Deleted: d117318e96f4acd3ec18cf20f5f2400a1ae82e0718a497e72aeecbe87d2ef119
Deleted: 133fc3bf0135487375244c22f13588512651dd7b4ad035871995990d299206e1
Deleted: 6985eb2bb1c56c344786880f28ba7e6f39e4a5181945ea65434b76a695308a04
Deleted: 882e335f1673165b9bc21e993ec5305cd09b0784c83ebb7ca29a729d94ab419f
Deleted: b0c69336b21d1a8144ea0c494da871b31bb6b81ce6b1096625d3240613371af6
Deleted: c9c11c37975c04ae782a1421c251b7a561d17aaa0eaf9422c14bdf4cf3406997
Deleted: f40eae475b7b30af06d09a9b369515cca4f954e58f55d6b1e39fc7ffa68c67bd
Error response from daemon: Conflict, cannot delete 86f47d28eef8 because the running container 1b84a98719f1 is using it, stop it and use -f to force
Deleted: 8f3045ed9d64d67f2454a6d889dd21c5d5f56b4ccf219b29cb0a0e81c21ac808
Deleted: 1480886714f9ca543c996baeb4a3f3bfebc2aa08ec365e508ffe25c059bc392d
Error: failed to remove images: [7aef41975329 2a2d0eebb7f7 c0a532310964 31170f0188d0 2e058bebddf2 b54a715ed91f c4e24794ae31]
core@esu-exp1-server-qhjozestjbvs ~ $ df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   673  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K  575K   66K   90% /var/lib/docker
```

ok... a little better... but not much... trying to build again:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker-compose build
mail uses an image, skipping
Building storage
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Step 0 : FROM busybox
 ---> 5356a35496ab
Step 1 : MAINTAINER Erik Kaareng-sunde <esu@enonic.com>
 ---> Using cache
 ---> 12bba6e83458
Step 2 : RUN mkdir -p /enonic-xp/home/repo
 ---> Using cache
 ---> ac41ff7557a5
Step 3 : RUN adduser -h /enonic-xp/ -H -u 1337 -D -s /bin/sh enonic-xp
 ---> Using cache
 ---> 46145d5c7df2
Step 4 : RUN chown -R enonic-xp /enonic-xp/
 ---> Using cache
 ---> 31ca57ee4181
Step 5 : VOLUME /enonic-xp/home/repo
 ---> Using cache
 ---> 539794b5d7dc
Step 6 : ADD logo.txt /logo.txt
 ---> Using cache
 ---> ce6eca23e41c
Step 7 : CMD cat /logo.txt
 ---> Using cache
 ---> 0cff1a09b6e6
Successfully built 0cff1a09b6e6
Building exp
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Step 0 : FROM enonic/xp-app:6.7.2
 ---> 2b04a1af0435
Step 1 : USER root
 ---> Using cache
 ---> 9e5e22d9a33b
Step 2 : RUN cp -r $XP_ROOT/home.org $XP_ROOT/home
 ---> Using cache
 ---> 5227574ec621
Step 3 : COPY deploy/* $XP_ROOT/home/deploy/
 ---> Using cache
 ---> 225c6e3b8638
Step 4 : COPY config/* $XP_ROOT/home/config/
 ---> Using cache
 ---> aac19299809f
Step 5 : COPY backup.sh /usr/local/bin/backup.sh
 ---> Using cache
 ---> 0e1991dd8a96
Step 6 : RUN chmod +x /usr/local/bin/backup.sh
 ---> Using cache
 ---> 84d2e01e29bc
Step 7 : RUN mkdir $XP_ROOT/home/repo
 ---> Using cache
 ---> 70746535a01e
Step 8 : RUN chown enonic-xp -R $XP_ROOT/home
 ---> Using cache
 ---> 9b3dafb3d3fb
Step 9 : COPY jolokia-jvm-1.3.5-agent.jar /srv/jolokia-jvm-1.3.5-agent.jar
 ---> Using cache
 ---> d45e6a25fbcf
Step 10 : ENV JAVA_OPTS "$JAVA_OPTS -javaagent:/srv/jolokia-jvm-1.3.5-agent.jar=port=1098,host=0.0.0.0,user=admin,password=admin"
 ---> Using cache
 ---> 959def4829a4
Step 11 : EXPOSE 1098 9200
 ---> Using cache
 ---> e40ca51fe425
Step 12 : USER enonic-xp
 ---> Using cache
 ---> 11be318c1797
Successfully built 11be318c1797
Building apache2
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Step 0 : FROM enoniccloud/apache2
latest: Pulling from enoniccloud/apache2
e20a84c3233b: Pull complete
d6c64ee0ce32: Pull complete
d8d3e3b9a2f4: Pull complete
ce76de2e871b: Pull complete
ba0539e66906: Pull complete
f0195a2798a0: Pull complete
e069a38e13dc: Pull complete
982fc9d225d0: Pull complete
32f84d442c35: Pull complete
be27f1c70f3a: Pull complete
80671ce96a73: Pull complete
2572def719e3: Pull complete
0a1c9297f0e3: Pull complete
50f6a4e785fa: Pull complete
a971625c4f60: Pull complete
5a99a0ae4955: Pull complete
101dd7ec4260: Pull complete
fa0e307a935c: Extracting [==================================================>]    874 B/874 B
51ee7b997886: Download complete
8a7c0765468d: Download complete
d4a22e425a33: Download complete
a93bb7fbd2bc: Download complete
7468977abb55: Download complete
Pulling repository docker.io/enoniccloud/apache2
e450b1758d1a: Error pulling image (latest) from docker.io/enoniccloud/apache2, ApplyLayer exit status 1 stdout:  stderr: open /usr/lib/perl/5.18.2/arybase.pm: no space left on device
8aa2fc7185e2: Error downloading dependent layers
ERROR: Service 'apache2' failed to build: Error pulling image (latest) from docker.io/enoniccloud/apache2, ApplyLayer exit status 1 stdout:  stderr: open /usr/lib/perl/5.18.2/arybase.pm: no space left on device
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
Damit!
ok.. removing all containers in docker-compose project:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker-compose rm
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Going to remove esuexp1_telegraf_1, esuexp1_exp_1, esuexp1_storage_1, esuexp1_mail_1
Are you sure? [yN] y
Removing esuexp1_telegraf_1 ...
Removing esuexp1_exp_1 ...
Removing esuexp1_storage_1 ...
Removing esuexp1_mail_1 ...
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
Removing esuexp1_telegraf_1 ... done
Removing esuexp1_exp_1 ... done
Removing esuexp1_storage_1 ... done
Removing esuexp1_mail_1 ... done
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   634  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K  638K  2.6K  100% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
hmm... need to delete images ( of course )...
Lets start with deleting all containers first..
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker ps -a
CONTAINER ID        IMAGE                                                              COMMAND                  CREATED             STATUS              PORTS               NAMES
a9ac45e8996a        d6aeba8e82d2e63648536de3369cf8dfaaddf1a46c60ce9fef2d85322ef070b4   "/bin/sh -c '#(nop) C"   19 hours ago        Dead
4e792a76d854        d6aeba8e82d2e63648536de3369cf8dfaaddf1a46c60ce9fef2d85322ef070b4   "/bin/sh -c '#(nop) C"   19 hours ago        Dead
8c9daf3eed84        d6aeba8e82d2e63648536de3369cf8dfaaddf1a46c60ce9fef2d85322ef070b4   "/bin/sh -c '#(nop) C"   19 hours ago        Dead
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker rm a9ac45e8996a 4e792a76d854 8c9daf3eed84
Error response from daemon: Cannot destroy container a9ac45e8996a: Driver overlay failed to remove root filesystem a9ac45e8996a787d6e1e9b54cd1328ae5ec427de748ede4c4fe49873002c8d9e: stat /var/lib/docker/overlay/a9ac45e8996a787d6e1e9b54cd1328ae5ec427de748ede4c4fe49873002c8d9e: no such file or directory
Error response from daemon: Cannot destroy container 4e792a76d854: Driver overlay failed to remove root filesystem 4e792a76d854170d15a226b2b0ea46d3a84de199172c04cbbeed11c0903b1e5f: stat /var/lib/docker/overlay/4e792a76d854170d15a226b2b0ea46d3a84de199172c04cbbeed11c0903b1e5f: no such file or directory
Error response from daemon: Cannot destroy container 8c9daf3eed84: Driver overlay failed to remove root filesystem 8c9daf3eed84cdace9f51e8e900cd6009406965d1b7a4a40fe0652f55a720a51: stat /var/lib/docker/overlay/8c9daf3eed84cdace9f51e8e900cd6009406965d1b7a4a40fe0652f55a720a51: no such file or directory
Error: failed to remove containers: [a9ac45e8996a 4e792a76d854 8c9daf3eed84]
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#

```
ok.. lets just close our eyes and force delete them:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker rm -f a9ac45e8996a 4e792a76d854 8c9daf3eed84
Error response from daemon: Cannot destroy container a9ac45e8996a: Driver overlay failed to remove root filesystem a9ac45e8996a787d6e1e9b54cd1328ae5ec427de748ede4c4fe49873002c8d9e: stat /var/lib/docker/overlay/a9ac45e8996a787d6e1e9b54cd1328ae5ec427de748ede4c4fe49873002c8d9e: no such file or directory
Error response from daemon: Cannot destroy container 4e792a76d854: Driver overlay failed to remove root filesystem 4e792a76d854170d15a226b2b0ea46d3a84de199172c04cbbeed11c0903b1e5f: stat /var/lib/docker/overlay/4e792a76d854170d15a226b2b0ea46d3a84de199172c04cbbeed11c0903b1e5f: no such file or directory
Error response from daemon: Cannot destroy container 8c9daf3eed84: Driver overlay failed to remove root filesystem 8c9daf3eed84cdace9f51e8e900cd6009406965d1b7a4a40fe0652f55a720a51: stat /var/lib/docker/overlay/8c9daf3eed84cdace9f51e8e900cd6009406965d1b7a4a40fe0652f55a720a51: no such file or directory
Error: failed to remove containers: [a9ac45e8996a 4e792a76d854 8c9daf3eed84]
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
ok... no more containers at least... Glad this wasn't in production... ( note to self: put metrics on inode usage ).
Now. Lets delete some images..
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
esuexp1_telegraf      latest              4147a25de438        19 hours ago        37.65 MB
esuexp1_exp           latest              11be318c1797        20 hours ago        1.251 GB
<none>                <none>              d6aeba8e82d2        25 hours ago        225.1 MB
esuexp1_storage       latest              0cff1a09b6e6        25 hours ago        1.096 MB
telegraf              1.0-alpine          facfea0a15fd        6 days ago          32.56 MB
enonic/xp-app         6.7.2               2b04a1af0435        2 weeks ago         1.251 GB
busybox               latest              5356a35496ab        2 weeks ago         1.093 MB
enonic/xp-app         6.7.1               c6a903727227        3 weeks ago         1.251 GB
<none>                <none>              101dd7ec4260        7 months ago        224 MB
enoniccloud/postfix   latest              595d658e6f11        12 months ago       287.2 MB
enonicio/apache2      latest              f723e20a3ab2        14 months ago       224 MB
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker rmi esuexp1_telegraf esuexp1_exp esuexp1_storage enonic/xp-app enonic/xp-app enoniccloud/postfix enonicio/apache2
Untagged: esuexp1_telegraf:latest
Deleted: 4147a25de438b42d78e9e1ba73203381050f8ba23352ed74d80c6997d0601953
Deleted: 86f47d28eef8632f8b84ab4024b5fec76ec4c8b59f51563b8c026fbd934a07d4
Untagged: esuexp1_exp:latest
Deleted: 11be318c17976190f6f0ad45deab93573c9a3a6e840a37fb7ec87c47e7798191
Deleted: e40ca51fe425f359a5cab88cced3be0a3cbefecf279621ff89420784ad3f7b28
Deleted: 959def4829a4467d88fdaa68012556d69f84564f113901712f0174d5f900c802
Deleted: d45e6a25fbcf379d844c09c8d2d572d16226e78a49278a3242027e93119801af
Deleted: 9b3dafb3d3fb13555001cfc6c46fa18618896731c37446d31325c3177145f8f0
Deleted: 70746535a01eb3dce1d49a7b45d08058f62a11f97b403bc72e994a5fc8324e37
Deleted: 84d2e01e29bcb489b647ae29cbf0b9f1e310d4d3d8e4aff009aecb5b02265bec
Deleted: 0e1991dd8a96d3b09c57b2bec5b477c1e45a046cad42d16a2069f3973a3dccd1
Deleted: aac19299809f8498b10fa6833d5a8b5ce4dc9ae9ebadc2ab824c9b188ccb0864
Deleted: 225c6e3b8638ee1376924299fb88e38066dff70cf2b080b67584b2febc4b4871
Deleted: 5227574ec6213b49920d47da5fbaa14a34f6308cace8c9837672a5a626e272a0
Deleted: 9e5e22d9a33b8e5c4d3825055b7c8707c4f7d4489a4dc12ba13b4c242c2f44f8
Untagged: esuexp1_storage:latest
Deleted: 0cff1a09b6e6cab4b00067335b37da92af93468106e153bc790ba0e2571013af
Deleted: ce6eca23e41c6dcf18a3911063eca505992a762dbb49fb463ef8ee535fe902b3
Deleted: 539794b5d7dc5ec83a769246817906b15a065e6296012c5b7ae751952508abc5
Deleted: 31ca57ee418117a565d6b3f01d50b1f03c60ab635cf9cbf70c9edf3c48cc5c30
Deleted: 46145d5c7df29eb85ab4d354f7eb7d3d0c3bc67d610ecba86978fe2de94f1056
Deleted: ac41ff7557a5f4356a5013794f0ef33ae5a0358bd61f1115be7237648944c2eb
Deleted: 12bba6e8345872d958744779afc1ea46751abd75ccd5b9a51ab76302f3e0767c
Error response from daemon: No such image: enonic/xp-app:latest
Error response from daemon: No such image: enonic/xp-app:latest
Untagged: enoniccloud/postfix:latest
Deleted: 595d658e6f1125ab1946e29546b94862db09b6ae954ffab816847156f0ada85e
Deleted: f32e14c939465ea3ac9c73fc8d833b7b227fb6bf28cf84feaa3181ae7e0a40a4
Deleted: 5345f3d69615fb4c128358d77eb548966f6c111ce08c3ead468f5806190f860a
Deleted: 3c931fe87f17575be0938624cf6ceec318e7138d4b669a7f298641ec65c54755
Deleted: b90db83ab65e085e03ea12be143450f6c2e65bd05d22d56c1924a9a98dd5a82d
Deleted: 0b88f96982adc98d03adf2f756d0a3b5fbe09c772a86f1e0978f23fcbd5e99eb
Deleted: fc2dcfc247b412d7b04c46307d231c08ceea004b27b6a9445ce1a00e6abfa042
Deleted: 170f00797a89fc90473a60f40c5e0e48b727166134ac1b8be7dcd841780f8e5c
Deleted: 6aec52b445e54607cc7e1b7a46a6353fcd05a4868cfc17ea2e4eb9a462dfc341
Deleted: c45b08b61931868f372b0007d5307987d2549573417b4568587975dbe6827f00
Deleted: 0e71605a3ea48f584eba5568dc9a6beb0a44c0fc09c5f7ccea13d835141381a3
Deleted: 5476ae4fa377c275454edeb51f9ae01d47de7032586cfb235627575e45b703b5
Deleted: 6033846807861478300e570defaabb1da782268560817200a34ffb679290d075
Deleted: b284696a3a3478243dcb464ec552122e4b5118d80d5912663bf4f1446941620b
Deleted: f317446ab54c002b4427cb6ef52a7ba63bf3fbc8634fbb4823ef85cd7fc61fba
Deleted: 6d668f14cda143d0a068d0908c7d894f465ddfc4f0b4831baefac7bc37b6e9b1
Deleted: 988c87b9dcbc3b50c380028b5abf0e66946905bba04401b472ffd33ea4961b41
Deleted: 0a17decee4139b0de68478f149cc16346f5e711c5ae3bb969895f22dd6723751
Deleted: 3c9a9d7cc6a235eb2de58ca9ef3551c67ae42a991933ba4958d207b29142902b
Deleted: eeb7cb91b09d5de9edb2798301aeedf50848eacc2123e98538f9d014f80f243c
Deleted: f9a9f253f6105141e0f8e091a6bcdb19e3f27af949842db93acba9048ed2410b
Untagged: enonicio/apache2:latest
Error: failed to remove images: [enonic/xp-app enonic/xp-app]
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              d6aeba8e82d2        25 hours ago        225.1 MB
telegraf            1.0-alpine          facfea0a15fd        6 days ago          32.56 MB
enonic/xp-app       6.7.2               2b04a1af0435        2 weeks ago         1.251 GB
busybox             latest              5356a35496ab        2 weeks ago         1.093 MB
enonic/xp-app       6.7.1               c6a903727227        3 weeks ago         1.251 GB
<none>              <none>              101dd7ec4260        7 months ago        224 MB
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   634  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K  497K  144K   78% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
Hmm... forgot to specify tag on some containers... *bah.!* ok, delete those images:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              d6aeba8e82d2        25 hours ago        225.1 MB
telegraf            1.0-alpine          facfea0a15fd        6 days ago          32.56 MB
enonic/xp-app       6.7.2               2b04a1af0435        2 weeks ago         1.251 GB
busybox             latest              5356a35496ab        2 weeks ago         1.093 MB
enonic/xp-app       6.7.1               c6a903727227        3 weeks ago         1.251 GB
<none>              <none>              101dd7ec4260        7 months ago        224 MB
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker rmi enonic/xp-app:6.7.2 telegraf:1.0-alpine enonic/xp-app:6.7.1 busybox
Untagged: enonic/xp-app:6.7.2
Deleted: 2b04a1af04357f96427190926712ad90e06866de458409fa9d74938c02be23a9
Deleted: 1aae025d3ee87c50514244e02ef0b37de206cda4278aa0d0e7f1556e2dc72cd7
Deleted: 3db83343f2461049671f59f6c5f42f402a82bf54dca4dab1d832f666ea4b9adf
Deleted: 939bf0db53bfa3e4fa7ee67a6d9c0008cabb61d7761db3948fc7f5cc064dc38e
Deleted: ca1f2a8245d604527659e57ab2dfe6bbb91bacbcc44e3f981aa784624bcf14ae
Deleted: c4d1201a76205b7fccd5cf80b30bac9a1b1d75f9dd810357c0860c64bebea6a8
Deleted: c24a9e0cecb32ba4f6dbfd0a05bfbdd4e7efbf537f44a6b187c96c2468586cd6
Deleted: b239c9ce6bd464f7c419782d5b6a1b4cc8592586b0d0950e5f896754ba61da85
Deleted: 5bc91b302970c547540af76d01a1cb3798826c0e5e336e6a18f505ea7482ab7a
Deleted: 95245c8c3d9eda05948be87e4324ca93899a5568975f825fdbfafb749432fc5c
Deleted: 2f34651c333b6e67ff528f1fc96a32f5f04d90b5c34e8f5bf1772061f79703fd
Deleted: 5247b4fa8d580a1a413efdcd129085dbe5e8d923bbe669fba842f17801a63110
Deleted: d40aedb5e4c1ce1e1979f375e87448ebdfdce4771af88e3f04b1893477dcaa5b
Deleted: dff08ea6e4698a21e0c1add4cbc858f509479c2889a43f4cfde59f96b547a95d
Deleted: a4654f0a2f4e2fb851858104375c9b06adefc3be6b4c3fee56e032b74a9a4c67
Deleted: 4340b07ca29c7418064a100c257a67f0b5b4558c0aa70a53011b4ef7bcf333c8
Deleted: 7eeb96559506864c35ec567463f13d6ebc7d8d7943ee19d661b2d2d61db2c931
Deleted: 03361774349e337160521cd51384b64763fc6b83012e94f297bff7cdfea65a22
Deleted: abe7b3906c90f188a0f795d886f21fc61f487cc721519395183370f40c566ada
Deleted: 4fbe1deb59bef21f8abb100c7626bd471c21c55668c6b8eddd72f2cbf9bb0cd5
Deleted: 072498340fd4330a6612cf0bae2a653257a72e4fdc40f47b77b2c26545e5cef5
Deleted: 91455eb72e1812ef59d6014d8019642fb566509a55004a6cc15bfafc6257231c
Deleted: a9fab23854cbcbe47515f77e0a822107584ec46c438493a33c121c2889f81a4b
Deleted: c02da7da5781aee9d7372f811b34bbe68e78969087c675f496389de2c598ecc0
Deleted: 27c96d3553fc02ed59a3826e72dedf46c248dafc17f37fe2664fdf5af592753d
Deleted: 656b9003330055a36e6c22f714ad955ecf2d46d2e6c7128e8cfa09a3ad447e01
Deleted: b11aa9ccf03bfa925b94b4679930584553f523097d6951bb52fd45e372f2f12b
Deleted: 70d1da65f07b4a87383962fedff2bee71e99e9c93de24d7fee2a15976477249e
Deleted: 1d492d5a025092f877ab70f70217a520c54334bf4e877fd56578bc2ac5c1c391
Deleted: 03398a59b6600eb15ac601e01e77e2567a4b87a2ed3e8cb92cbd3e75f4b646ec
Deleted: aa3d89c4db732d22be9e57dd7cc13a5927db3077b235ebbe37a31a06b33fe03d
Deleted: 8b84ec62838fafece3e9cd490f2d28187b80c23eae54294235a29293c028929a
Deleted: 4fdc3ad74ef18ca5c4eaef3586203171c4538ed6d0831530639da7b3067fbe99
Deleted: 4e3fa3e2df601bf104d0ffb58089d8df869571d9ffd8acbbbc1b5755f85bfe9d
Deleted: 2a3772cc35dd414b0ff002f4883103ed8d1b501e0b2ec6106899a5ada52e0f0b
Deleted: 19cd5e0c8bf422786d1ecd242e69f7155c3cc87b3b05e7611f01e974e32e6063
Deleted: e7d16eb4beda6e9bb8d61ca47c8ef5e07ab1f9af0ea1f179475704ef06e99c9a
Untagged: telegraf:1.0-alpine
Deleted: facfea0a15fd3263d609c0cab51fdb977e940e49be64f960d051fdf3bac068a1
Deleted: ba39cc3f7bbf4cc6afca798ad97f2b671113f5e4ee2370925c9ae4bfcae4aa4c
Deleted: 2e6754b4bf4316c6f3d7dec64faa17604ddfe1cd01312b991db532d7d52947e9
Deleted: dd8c22c3be3d4be7bad2db833202efb6886982233b032a10aabae5f90540bbfb
Deleted: c50bac2990c1104fae0b4794edc6eaef7535706d648abca139aaac01cea98ede
Deleted: b1bfcd9b8984ead0cc2428be851697d098fc59bd6d5511f112c357b18d71d6ba
Deleted: 30775f4d77f791bb79a2ed0d48e6d4ffb2691529e23ef0e4fc4f380c01797ef2
Deleted: 20119b0aa035c9a0f607cb22bc3aa0e41c4629e4e86e44c8ef30cf3d07dc7d89
Untagged: enonic/xp-app:6.7.1
Deleted: c6a90372722707b2260be85326d7e1a9953552c2e519f93d3d5692c65d8827c3
Deleted: dc784505b372904f7fa1745aa5e6fcf98454721df642bd3d58225e008f4c8ab1
Deleted: f8ef69b945f522627fdc6ed494be988b2c8422227b34331eec25178867a123db
Deleted: 98909723f31faf7f93c843d51ec247392ad9fc621df7f37cdf4b32a668f3fed2
Deleted: 7de32ac4b7d99a9b90dc5056d56c2eff7aa1bd569d050d7efb95412b466c114c
Deleted: 3c68367f8c621b69f8483f87b34cacad6ec89a3bf87781ba89dae58fa381df7f
Deleted: df4be444aea26736081aa9732c5823417cb8d3b0ad33ed3e77daa3f48f4d5ed8
Deleted: 0616e914fcc8781637ea706280f3c83820d820bbe2a66f927b958d4535079091
Deleted: fdf00b408a280efb0eab71ab8a275a4229ebdade2176ea8fa12f98ec0d63177a
Deleted: ecc11a4ebce2d1c2b94167f5baba8dc97e2b2535dbad19cfbe353bd6aa5d645d
Deleted: bcd8cd454f146f6a217dfc123eda83b95a6d1a78d2bb6180e9514ab390eeb28d
Deleted: 93861882055657eed6b88dc0f77805e7f4f2828571a58b68ae3a1177ae5e6e68
Deleted: 306e9c5664fc37ac2dc7d35005313d6b8078ae64034d01b159ba78cea6e54cfc
Deleted: c6d34831e6f5ffd1048e6af7f7c7a9955e428731589c8617bf199965cf03f58f
Deleted: 040e3bda35cfbe98b93b29f780d8167666659728b0dc84535656db36e9202384
Deleted: 5561d172e82700814dfa48417b07cdaa98393dddf9e09466c2c3a48eaf061ca0
Deleted: 1349ca6e91ff4edb1a2bb51acd50020d9e24c9784f5840723f9988cb16c05c2d
Deleted: 21a88a8176dc78dab50b452b2b4029fd5fd66d46322f7eac71efc92c472cd09c
Deleted: 5ff3adf81c412ec43f1103b1472f1132ccee9a1f61e8a3e6e414c51f55a00a04
Deleted: cc137f7c497369b53d9a1e6984755b143a9a7f958ed87e9132f1aab87916debb
Deleted: f69c2a40b05ace9aee54ddb03ded2a542df20387126df76bf83f3ac404d58c58
Deleted: 8e2b79bee8ec564369d5955755e159488380a2e93991f664f824376a4c3bd031
Deleted: 833fd71c2655732a15e1ad70b9c62fcaeebdf37fe0e61605b792923799f20769
Deleted: 3ee38c36654626a7fa3294b59004747f3ccb4a9fd971beec0e106b4a34e8e633
Deleted: bb7202f980cc78e0a15ed91df0bfd61597ae945cde6a2d32e0627040c01d9af8
Deleted: 3d8b86e2d55fd3c3596773eb35cd6643cd4971cce6635c777b165c9b8d15374b
Deleted: 39a7985831a2cb23df2674761fc9297bcdb14fb99b8b9b7071af7e57cc0397c4
Deleted: a94031acea35d06ad4a4d357c1b743d8a728eddce331a28ba820f1b5b8ccdd16
Deleted: 83e5681d48d4d0b849aa4d9e1598fbbf5b4ecd9592ef3f3a399e9a60e5b318f1
Deleted: 282076803beec128807b72fb768ca5ca3ff05e1cee963631f689d1f27ef897d3
Deleted: 9c389fdd7e4b4e4754b4a5aa398ef3541554d31b828d76b1eef8300601856cca
Deleted: a282ed9383fe31077ab0158a0eb57e421a0f37b37afe3df949e634e259721044
Deleted: 771ebee7e1d61a855251c793ecf3b28c7c1bb07bc083c56cb17221f817cde12d
Deleted: fca9842eed3bba72d897573e168492d519a3c5b68ded228e7fabafad9b6471b2
Deleted: 49dd2639f992346d3c9a6a28558355276cf9814996bbe77e9aefc5aee787508b
Deleted: 5c2c4906b7c1c6b6d2bb2e6993439469a9958c339877aa2aee0b6592fa0071c7
Deleted: 845b177c8afc71e9bc42eb99d9191bd6df14de7d41cb62dea5671724c4a17384
Deleted: 348597ef985689f4fc6a4114ebf4882eeae35d451e92f2e733bf3c0d139f0725
Deleted: 36377ffedd47d8fc48d9bd6f6e2e9bd10d88cf56374a800b04241b629a775d26
Deleted: 620c155c396d774940a2fcbaa6d9936224401b85f69e818ee18706eb6f7742f7
Deleted: 23adddc76d9f3dafaaaedb5ffc0449a346a79f60a1f0de1b7222e385d5e0600d
Deleted: cc4bd1ce7aa5ef02d8abf133b01d582b940feea9b39b3db571eb967e6725d521
Deleted: 1eaa89fafe9d9975b4aa566b70827ba1df64f9ca14e21b371d0f5d3769e0649e
Untagged: busybox:latest
Deleted: 5356a35496ab68a1f4fbf80060b3b04153dc1f00ab4e28f41443d9955bf675bc
Deleted: 363a10951ae2846a667fe98bcda6a9c023bc20f82b362dd08277d11bb8e8a9b6
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
 Lets  see...
 ```
 root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              d6aeba8e82d2        26 hours ago        225.1 MB
<none>              <none>              101dd7ec4260        7 months ago        224 MB
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   634  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K  149K  492K   24% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
ok.. so still to untaged images and they are using 24% of the inodes... lets delete them
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker rmi d6aeba8e82d2 101dd7ec4260
Deleted: d6aeba8e82d2e63648536de3369cf8dfaaddf1a46c60ce9fef2d85322ef070b4
Deleted: f723e20a3ab27d26ad436545863c8d998d18d9537d884cb1bca34d8cebb0eb13
Deleted: c3c5b48831f823c7b34ea79a064ee5e83e2305ef7255cf10bdd1bdd5cea4b0d5
Deleted: 73291610ea96c7c55a712799ed42d9fa935deb0c7e805599afc4e45acb2c9a1f
Deleted: d509c11a646c842e6ef2096d955ba4e68c2dbed1ec2dc92cf04b56dd0d9cc0d9
Deleted: 58672edcfb1db4e05efc242dd39202a5a3ace06043a8e9b62e711901f5c6c777
Deleted: 15e49b7da13682dbe34e1aa056ac3df85577c550324d9fe827c627a36dd06d2d
Deleted: 8c80bbafc72003eb98a9ede46e17e2d878319f0db5bca9a9cbff878a818ea1c6
Deleted: 34b11a87f1bc5d27346a5c68db1c7094953170ccb6766de2fe0f48cba475ce38
Deleted: d3b0c7b59443bc51158640033ff8029a69be8f4122f7d520f9a0e4dfb492d9c6
Deleted: 69a2b67c0f4fdbf331b4b09cc7da6f8c25544236bd0b3a0089905f70fe8e276d
Deleted: 5655484d6f5297e1fb0468a630c6b981ca8fe08c828dfeb35c84f703eedb61d4
Deleted: 66622ad295bfc73ea93b1142984d55d894cd50fb59ad09eb47dc0dce8efe8bed
Deleted: b73268879210886a21b56a1fbc022b0c58402fabf55764d189e19c72480f176e
Deleted: a3fb3aca6dd0b4886344b88676a47e1b7105b287ebac34a1456ccb6adf0c099c
Deleted: 7e8910651a41512beccf0e57cab7c7872b9310bb493958d20e84d577d82cc370
Deleted: a5d2cd258a783930de760d8805877cba65f78fdc88b3c08eabc0c2776d921a25
Deleted: 4538a0bdde26f8dd4d80f46ffd73472a42317a0d82ed0335ba1e0aa9ad5321d5
Deleted: d5f579084c5612e83bb2396b60fbc12a9502bde5460eb51559c05116fdf27d9b
Deleted: d75fcad85ca85526358cafa48639e4b4338aa0a0c1e887e4ffe2c92d862c978b
Deleted: 89bc2eaf6fe4ce65ea76792a6117805ef84dfa243dbc190e1e464b88b9f2e6f9
Deleted: 18c0be32c89644d485620cfe6268bf2da760c248403ce1b71bf31a4136f39dd5
Deleted: de9f9f30df912ba59ef8d9e286e23f07f6824d1dd65a5149f7f55a181fc133b7
Deleted: 5b76f7c1436d8d0de4ba250d699e71c4a5f1fe7ac0296bc3db8ed4cca402cf8c
Deleted: 101dd7ec4260ff34dac97bbb873348db14e2b7a19a652ec4d5d3466e733df80c
Deleted: 5a99a0ae495567822a9dd581d09eb8b264ab540138c9698027736d70e0d525ce
Deleted: a971625c4f6092da659f40fc0bf18b7a6125742aca8e1a416dc77c3a774531f0
Deleted: 50f6a4e785faf4b6aeb29136c511b266a8895a822bb3c1f3c1ad06757d5d319c
Deleted: 0a1c9297f0e3df0a4def3100dc0ed3a926c19b09740b6d3c48ac951aabee0c56
Deleted: 2572def719e3a8661566a1c1b0420048051c9ae744d255a30d89e2d861fc6435
Deleted: 80671ce96a733aaa1a184951ef4dc5b1529d45581078325b32afafc009d0f067
Deleted: be27f1c70f3a64e8e15ce0a78a3512949c78d4eefa1724f83413f8441cf06be9
Deleted: 32f84d442c355710413eef946e23e295d903c731ee2acf185cc983ff456edcfd
Deleted: 982fc9d225d0810b1b8efbbf16d3bb8325fe23d2b1747edaace08bb979cf7c83
Deleted: e069a38e13dc9fdf7f1d0856aee8a7516ad5586b406bbc869152f8b342ffcc73
Deleted: f0195a2798a0e75fa2812125878351ed880d7b6028799ebad5422c09c37bed71
Deleted: ba0539e669067ceaa68c7f671d55bc97d0f3324961e81d5c911db208b3f4610b
Deleted: ce76de2e871b712eec51f0fe38fed4946d483bf5c85c9e771d9a5a90f4386066
Deleted: d8d3e3b9a2f454328b29cabb46733ca60ce692eaaf6b6cc9977a91d76c441e31
Deleted: d6c64ee0ce3258f5b87ebb787d182e5307224c1215ca71547e5e15b6f447b1fc
Deleted: e20a84c3233b107e0a41d119cbdcf651d571fb0aabda65e50db7e0e1f39eb03f
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   634  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K   413  640K    1% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
ok.. back to square one... lets pull down each image one by one and se which image is failling...
Starting with the apache2 base image:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# cat apache2/Dockerfile
FROM enoniccloud/apache2

COPY var-www-html /var/www/html
COPY sites /etc/apache2/sites-enabled

RUN a2enmod proxy_wstunnel
RUN a2enmod proxy_http
RUN a2enmod rewrite

run rm /etc/apache2/sites-enabled/000-default.conf
run rm /etc/apache2/sites-enabled/default-ssl.conf

EXPOSE 8001
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker pull enoniccloud/apache2
Using default tag: latest
latest: Pulling from enoniccloud/apache2

e20a84c3233b: Pull complete
d6c64ee0ce32: Pull complete
d8d3e3b9a2f4: Pull complete
ce76de2e871b: Pull complete
ba0539e66906: Pull complete
f0195a2798a0: Pull complete
e069a38e13dc: Pull complete
982fc9d225d0: Pull complete
32f84d442c35: Pull complete
be27f1c70f3a: Pull complete
80671ce96a73: Pull complete
2572def719e3: Pull complete
0a1c9297f0e3: Pull complete
50f6a4e785fa: Pull complete
a971625c4f60: Pull complete
5a99a0ae4955: Pull complete
101dd7ec4260: Pull complete
fa0e307a935c: Pull complete
51ee7b997886: Pull complete
8a7c0765468d: Pull complete
d4a22e425a33: Pull complete
a93bb7fbd2bc: Pull complete
7468977abb55: Pull complete
Digest: sha256:c54214989c66806cceb507377df648e8cd30233187f142f7972eef7e95cfac35
Status: Downloaded newer image for enoniccloud/apache2:latest
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   634  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K   83K  558K   13% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
ok... så 12% inode usage on one image... lets se How big the other images are....
Fist off, the Enonic XP image:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   634  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K   83K  558K   13% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker pull enonic/xp-app:6.7.2
6.7.2: Pulling from enonic/xp-app
1eaa89fafe9d: Pull complete
cc4bd1ce7aa5: Pull complete
23adddc76d9f: Pull complete
620c155c396d: Pull complete
36377ffedd47: Pull complete
348597ef9856: Pull complete
e7d16eb4beda: Pull complete
19cd5e0c8bf4: Pull complete
2a3772cc35dd: Pull complete
4e3fa3e2df60: Pull complete
4fdc3ad74ef1: Pull complete
8b84ec62838f: Pull complete
aa3d89c4db73: Pull complete
03398a59b660: Pull complete
1d492d5a0250: Pull complete
70d1da65f07b: Pull complete
b11aa9ccf03b: Pull complete
656b90033300: Pull complete
27c96d3553fc: Pull complete
c02da7da5781: Pull complete
a9fab23854cb: Pull complete
91455eb72e18: Pull complete
072498340fd4: Pull complete
4fbe1deb59be: Pull complete
abe7b3906c90: Pull complete
03361774349e: Pull complete
7eeb96559506: Pull complete
4340b07ca29c: Pull complete
a4654f0a2f4e: Pull complete
dff08ea6e469: Pull complete
d40aedb5e4c1: Pull complete
5247b4fa8d58: Pull complete
2f34651c333b: Pull complete
95245c8c3d9e: Pull complete
5bc91b302970: Pull complete
b239c9ce6bd4: Pull complete
c24a9e0cecb3: Pull complete
c4d1201a7620: Pull complete
ca1f2a8245d6: Pull complete
939bf0db53bf: Pull complete
3db83343f246: Pull complete
1aae025d3ee8: Pull complete
2b04a1af0435: Pull complete
Digest: sha256:1287630a2fe928611b82dffe57abc04d927592bfb3199ee9455c60d5ff2582ab
Status: Downloaded newer image for enonic/xp-app:6.7.2
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   634  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K  268K  373K   42% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
ok.. so 19% inode usage for that image.. thats not good... :-$
If you see compared to how much disk space we use:
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        989M     0  989M   0% /dev
tmpfs          1003M     0 1003M   0% /dev/shm
tmpfs          1003M  1.4M 1002M   1% /run
tmpfs          1003M     0 1003M   0% /sys/fs/cgroup
/dev/vda9       7.4G  560M  6.5G   8% /
/dev/vda3       985M  492M  442M  53% /usr
/dev/vda1       128M   67M   61M  53% /boot
tmpfs          1003M     0 1003M   0% /media
tmpfs          1003M     0 1003M   0% /tmp
/dev/vda6       108M   60K   99M   1% /usr/share/oem
/dev/vdb1       9.8G  2.0G  7.3G  21% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1#
```
We are eating up the inode table 2 times as fast as our disk space. We only have 10GB of storage here, so the number of inoides are limited and therefore we see the problem much faster than on larger partitions.

Let's see how many inodes we use when we do a full `docker-compose build`..
```
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# docker-compose build
mail uses an image, skipping
Building storage
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Step 0 : FROM busybox
latest: Pulling from library/busybox
363a10951ae2: Pull complete
5356a35496ab: Pull complete
Digest: sha256:98a0bd48d22ff96ca23bfda2fe1cf72034ea803bd79e64a5a5f274aca0f9c51c
Status: Downloaded newer image for busybox:latest
 ---> 5356a35496ab
Step 1 : MAINTAINER Erik Kaareng-sunde <esu@enonic.com>
 ---> Running in 70fd8b1509e4
 ---> ae9cc3dee516
Removing intermediate container 70fd8b1509e4
Step 2 : RUN mkdir -p /enonic-xp/home/repo
 ---> Running in deb33246074f
 ---> 950c75f08d34
Removing intermediate container deb33246074f
Step 3 : RUN adduser -h /enonic-xp/ -H -u 1337 -D -s /bin/sh enonic-xp
 ---> Running in ec906c607843
 ---> ef193eda895c
Removing intermediate container ec906c607843
Step 4 : RUN chown -R enonic-xp /enonic-xp/
 ---> Running in 12880ad32f76
 ---> 61a760072995
Removing intermediate container 12880ad32f76
Step 5 : VOLUME /enonic-xp/home/repo
 ---> Running in 05d430247a1c
 ---> e42ceffe4f87
Removing intermediate container 05d430247a1c
Step 6 : ADD logo.txt /logo.txt
 ---> 3e417ffab7bb
Removing intermediate container 0517ed4f4b6f
Step 7 : CMD cat /logo.txt
 ---> Running in 79ba6668cf48
 ---> d739ece36053
Removing intermediate container 79ba6668cf48
Successfully built d739ece36053
Building exp
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Step 0 : FROM enonic/xp-app:6.7.2
 ---> 2b04a1af0435
Step 1 : USER root
 ---> Running in 8db0abae004a
 ---> 3bcbe09e2447
Removing intermediate container 8db0abae004a
Step 2 : RUN cp -r $XP_ROOT/home.org $XP_ROOT/home
 ---> Running in 7cd799dc8e3c
 ---> 65a8e7ae04bd
Removing intermediate container 7cd799dc8e3c
Step 3 : COPY deploy/* $XP_ROOT/home/deploy/
 ---> e96cf26205bf
Removing intermediate container 100d92726bc5
Step 4 : COPY config/* $XP_ROOT/home/config/
 ---> f6d11604cecf
Removing intermediate container c6e12e34264c
Step 5 : COPY backup.sh /usr/local/bin/backup.sh
 ---> 601f2056e916
Removing intermediate container a6e2c61e893a
Step 6 : RUN chmod +x /usr/local/bin/backup.sh
 ---> Running in 1e2f8d513be8
 ---> 4e4fb8ec4dd0
Removing intermediate container 1e2f8d513be8
Step 7 : RUN mkdir $XP_ROOT/home/repo
 ---> Running in 98d785c9314e
 ---> 36fd3dba9d25
Removing intermediate container 98d785c9314e
Step 8 : RUN chown enonic-xp -R $XP_ROOT/home
 ---> Running in b3a7e056ffff
 ---> be973056fb02
Removing intermediate container b3a7e056ffff
Step 9 : COPY jolokia-jvm-1.3.5-agent.jar /srv/jolokia-jvm-1.3.5-agent.jar
 ---> a2cfee07f927
Removing intermediate container 2167f77094fe
Step 10 : ENV JAVA_OPTS "$JAVA_OPTS -javaagent:/srv/jolokia-jvm-1.3.5-agent.jar=port=1098,host=0.0.0.0,user=admin,password=admin"
 ---> Running in e6cc1dbe65aa
 ---> 305481afe485
Removing intermediate container e6cc1dbe65aa
Step 11 : EXPOSE 1098 9200
 ---> Running in 7760c115bbf8
 ---> 6864f3fc98d8
Removing intermediate container 7760c115bbf8
Step 12 : USER enonic-xp
 ---> Running in b8f0d75230c0
 ---> 3e22acf8e0ca
Removing intermediate container b8f0d75230c0
Successfully built 3e22acf8e0ca
Building apache2
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Step 0 : FROM enoniccloud/apache2
 ---> 7468977abb55
Step 1 : COPY var-www-html /var/www/html
 ---> 2a2f25c7d1ff
Removing intermediate container 7dd2dcf61beb
Step 2 : COPY sites /etc/apache2/sites-enabled
 ---> 6a825e97b673
Removing intermediate container c0fbb112fec2
Step 3 : RUN a2enmod proxy_wstunnel
 ---> Running in d54f2734595a
Considering dependency proxy for proxy_wstunnel:
Enabling module proxy.
Enabling module proxy_wstunnel.
To activate the new configuration, you need to run:
  service apache2 restart
 ---> ebf2cf6efd86
Removing intermediate container d54f2734595a
Step 4 : RUN a2enmod proxy_http
 ---> Running in 7fa359b31255
Considering dependency proxy for proxy_http:
Module proxy already enabled
Enabling module proxy_http.
To activate the new configuration, you need to run:
  service apache2 restart
 ---> ff019367e35b
Removing intermediate container 7fa359b31255
Step 5 : RUN a2enmod rewrite
 ---> Running in aa935b2d45e5
Enabling module rewrite.
To activate the new configuration, you need to run:
  service apache2 restart
 ---> 11d84848a699
Removing intermediate container aa935b2d45e5
Step 6 : RUN rm /etc/apache2/sites-enabled/000-default.conf
 ---> Running in 4f076267e54e
 ---> d3ee0875ca0c
Removing intermediate container 4f076267e54e
Step 7 : RUN rm /etc/apache2/sites-enabled/default-ssl.conf
 ---> Running in cc3a9e784085
 ---> 99a2636a167f
Removing intermediate container cc3a9e784085
Step 8 : EXPOSE 8001
 ---> Running in bcfc2114834f
 ---> e711c954ed98
Removing intermediate container bcfc2114834f
Successfully built e711c954ed98
Building telegraf
/python/local/lib/python2.7/site-packages/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
Step 0 : FROM telegraf:1.0-alpine
1.0-alpine: Pulling from library/telegraf
20119b0aa035: Pull complete
30775f4d77f7: Pull complete
b1bfcd9b8984: Pull complete
c50bac2990c1: Pull complete
dd8c22c3be3d: Pull complete
2e6754b4bf43: Pull complete
ba39cc3f7bbf: Pull complete
facfea0a15fd: Pull complete
Digest: sha256:406017b0a83f61032ca4fcf7f1787d83910f43ee5c1f3e68b2e5bc7bdbdb3762
Status: Downloaded newer image for telegraf:1.0-alpine
 ---> facfea0a15fd
Step 1 : RUN apk update && apk add net-snmp-tools
 ---> Running in a3c9802fc75b
fetch http://dl-cdn.alpinelinux.org/alpine/v3.4/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.4/community/x86_64/APKINDEX.tar.gz
v3.4.4-45-g3c26567 [http://dl-cdn.alpinelinux.org/alpine/v3.4/main]
v3.4.4-21-g75fc217 [http://dl-cdn.alpinelinux.org/alpine/v3.4/community]
OK: 5973 distinct packages available
(1/3) Installing net-snmp-libs (5.7.3-r5)
(2/3) Installing net-snmp-agent-libs (5.7.3-r5)
(3/3) Installing net-snmp-tools (5.7.3-r5)
Executing busybox-1.24.2-r11.trigger
OK: 10 MiB in 15 packages
 ---> 65ba1df2c1c7
Removing intermediate container a3c9802fc75b
Step 2 : COPY telegraf.conf /etc/telegraf/telegraf.conf
 ---> 79a672659603
Removing intermediate container 903bcb72e164
Successfully built 79a672659603
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         248K   330  247K    1% /dev
tmpfs            251K     1  251K    1% /dev/shm
tmpfs            251K   660  251K    1% /run
tmpfs            251K    14  251K    1% /sys/fs/cgroup
/dev/vda9        2.0M   672  2.0M    1% /
/dev/vda3        254K  6.4K  248K    3% /usr
/dev/vda1           0     0     0     - /boot
tmpfs            251K     1  251K    1% /media
tmpfs            251K     9  251K    1% /tmp
/dev/vda6         32K    13   32K    1% /usr/share/oem
/dev/vdb1        640K  356K  285K   56% /var/lib/docker
root@4d9a828891b7:/srv/home/Downloads/esu-exp1# ssh -i /python/src/database/projects/$EC_ID/ssh/private.key core@185.56.187.230 df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        989M     0  989M   0% /dev
tmpfs          1003M     0 1003M   0% /dev/shm
tmpfs          1003M  1.5M 1002M   1% /run
tmpfs          1003M     0 1003M   0% /sys/fs/cgroup
/dev/vda9       7.4G  560M  6.5G   8% /
/dev/vda3       985M  492M  442M  53% /usr
/dev/vda1       128M   67M   61M  53% /boot
tmpfs          1003M     0 1003M   0% /media
tmpfs          1003M     0 1003M   0% /tmp
/dev/vda6       108M   60K   99M   1% /usr/share/oem
/dev/vdb1       9.8G  2.2G  7.1G  24% /var/lib/docker
```
ok.. so not as bad as I tought, but still we are eating up 56% of the inodes just by installing the service...



# Solution
- Increase inodes?

- Slim down images?

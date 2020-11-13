## Indices in a File Image
Since our math indexer creates a large amount of directories and
files on disk proportionally to the number of expressions you are
going to index.
If you are using a typical file system on Linux, e.g. EXT4, you are
limited to create so many directories/files by the number of inodes
on your file system.

To overcome this problem, as well as improve efficiency of math index,
it is better to create a disk image as a loopback device to be
partitioned by some file systems which do not put restriction on inodes.
The file system should be efficient in crucial aspects of benchmarks that
have great impact on search performance. Later, we can simply mount this
disk image to be used as our index "disk".

After some investigation, we choose to use ReiserFS as our default index
file system due to its overall fast sequential read and random seek
(see
[ref1](http://girlyngeek.blogspot.com/2011/04/ultimate-linux-filesystems-benchmark.html)
and
[ref2](https://debian-administration.org/article/388/Filesystems_ext3_reiser_xfs_jfs_comparison_on_Debian_Etch)
).

To create, mount and unmount a ReiserFS disk image, we provide a few simple
scripts located under `$PROJECT/indexer/scripts`. Creating and mounting a
disk image are simple as follows:

```sh
$ cd $PROJECT/indexer
$ ./scripts/vdisk-creat.sh reiserfs
$ sudo ./scripts/vdisk-mount.sh reiserfs
```
A `vdisk.img` is created as our ReiserFS disk image, and is mounted to
`./tmp` so we can just use indexer and searcher on `./tmp` like a
normal directory.

After you are done, unmount `./tmp`:
```sh
$ sudo ./scripts/vdisk-umount.sh
```

### Some notices

### 1. Lacking kernel support for ReiserFS support
If you are running on kernel without ReiserFS support, modify scripts 
argument above and change file system to `btrfs` for similar performance.

For server distributions support ReiserFS, Debian 8.5 is recommended one.
On Debian, install `reiserfsprogs` for userland ReiserFS supports (e.g. mkfs).
```sh
$ apt-get install reiserfsprogs
```

### 2. `dd` command reports exhausted memory
When you experience `dd` command not being able to create certain size of image file:

```sh
dd: memory exhausted by input buffer of size 1073741824 bytes (1.0 GiB)
```
Try either reduce the `bs` argument number of `dd`, or use a disk swap file:
```
dd if=/dev/zero of=/swapspace bs=1M count=4000
mkswap /swapspace
swapon /swapspace
```

### 3. TRIM in SSD
If you are doing indexing on an SSD drive (which is recommended because it is often more than 4 times faster than hard disk in terms of random write performance), it is highly suggested to enable SSD TRIM whenever it is supported, due to SSD Write Amplification (WA) effect. Without TRIM, the intensive writing onto SSD drive can cause very slow indexing performance and reduce SSD life span.

TRIM can be invoked either continuously by mounting your SSD drive with `discard` option
```sh
sudo mount -o discard,noatime /dev/nvme0n1 ./mount‑point
```
where `noatime` option stops to record the timestamp of accessing files (and directories) to further reduce the number of writing operations performed on SSD.

Or, by periodically run `fstrim` command:
```sh
$ sudo fstrim -v ./nvme0n1/
```
alternatively, The util‑linux package provides `fstrim.service` and `fstrim.timer` systemd unit files. Enabling the timer will activate the service weekly:
```sh
$ sudo systemctl enable fstrim.timer
$ sudo systemctl start fstrim.timer
$ journalctl ‑‑unit fstrim.timer # show logs
```
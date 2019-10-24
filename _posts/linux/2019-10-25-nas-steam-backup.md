---
title: Downloading Steam Games from NAS
category: linux
published: true
tags: ['nas', 'samba', 'cifs', 'steam', 'gaming', 'windows', 'zfs']
description: Using Steam's backup/restore feature and ZFS to speed up game downloads.
comments: false
date: 2019-10-25
---

The amount of free space on my NAS made me think about other things I could be doing with it. I came up with optimizing Steam downloads:<br/>
Since I only have 2 256GB SSDs in my gaming rig, it is fairly space-limited for modern games. So I figured I could store games that I'm not currently playing on my NAS, removing the need to re-download them or buy spinning rust disks for the gaming rig.

### Creating a network share

This is really easy thanks to ZFS:

{% highlight bash %}
sudo apt install samba
sudo zfs create -o exec=false datapool/games
sudo chown bennett:bennett /datapool/games

# add a new smb user and set his password
sudo smbpasswd -a bennett
# enable network sharing
sudo zfs set sharesmb=on datapool/games
{% endhighlight %}

To save space, I also turned off automatic ZFS snapshots for this dataset:

{% highlight conf %}
# /etc/sanoid/sanoid.conf

[datapool/games]
    use_template = never

# ... <snip> ...

[template_never]
    hourly = 0
    daily = 0
    monthly = 0
    yearly = 0
    autosnap = no
    autoprune = no
{% endhighlight %}

### Backing Up Steam Games

Steam has a backup/restore option in the `File` menu. It would not let me choose network locations, and I have zero experience with Windows, but a quick google revealed that Windows can mount (they call it `map`) network shares as a new drive.

1. Open the list of network shares
2. Access `\\Nas\datapool_games` and log in to make sure it works
3. Right-click on the share and select `map network drive`. I chose `\Z:` as target.
4. In Steam, create a new backup in `Z`.
5. Profit.

This will allow me to restore games at the sequential read speed of the ZFS pool! Since it is a mirror of 2 WD Reds connected via Gigabit LAN, their combined speed of `~2x100MB/s` is limited by the ethernet connection. This should give me speeds over 10 times faster than my fiber connection!
By the way, Steam appears to compress the backups, so whether compression is enabled in ZFS is irrelevant.

### Benchmark

Since I am a computer scientist, let's have a look at what speed I get in practice:

<figure>
    <img src="https://vps1.piater.name/img/steam-restore.png">
    <figcaption>Peak Restore Speed through SMB/CIFS over gigabit ethernet</figcaption>
</figure>

`859 Mbps` is pretty good, isn't it? That's about `107 MBps`, or around 9 seconds to transfer a Gigabyte.<br/>
It is also close enough to the maximum data throughput of IEEE 802.3ah (Gigabit Ethernet) that the remaining overhead is probably due to SMB overhead --- it is known to be quite bad.
<br/>Either way, this gives me 11.5 times more speed than my 75mbps fiber connection, so it was clearly worth it!
I kind of which I had 10Gbps LAN now though, but that will have to wait another 5 years or so at least...

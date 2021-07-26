# Warrior VM for TrueNAS

auto-updating Warrior VM based on [ClearLinux](https://clearlinux.org/) and the [ArchiveTeam Warrior Docker Image](https://github.com/ArchiveTeam/warrior-dockerfile)

(you can probably use this image on other hypervisors too, as long as they provide a UEFI firmware, a VirtIO disk and a VirtIO NIC)

## system facts

* hostname: `warrior`
* username: `warrior`
* password: `archiveteam`
* timezone: `UTC`
* keyboard layout: `us`

You can change all of these via the normal linux commands.
Changes to the timezone are propagated to the warriors after they are restarted.

### enabled services

* swupd automatic updates
* sshd (no root login allowed)
* docker

### DNS

The VM also runs a small [unbound](https://nlnetlabs.nl/projects/unbound/about/) container as hardcoded DNS resolver for the warriors to circumvent any DNS filtering/blocking done by you, your ISP or your government.¹

This container is not reachable from outside of the VM.

## setup on TrueNAS

* create a 64 GiB zvol²
* download the [VM image](https://github.com/HorayNarea/warrior-vm/releases) and verify it's checksum
* "burn" the image onto your zvol by doing `zstd -d -c $imagefile|dd bs=1m of=/dev/zvol/…`
* create a VM
    * Guest Operating System: Linux
    * System Clock: UTC
    * Boot Method: UEFI
    * at least 1 GiB RAM
    * recommended vCPU Cores: 2 or 4
    * Use existing disk image
        * Select Disk Type: VirtIO
        * select your previously created zvol
    * Network Adapter Type: VirtIO
* boot the VM
* open http://warrior:8000 and http://warrior:8001 in the browser to monitor your warriors

**If you change anything in the warrior web configuration, your changes will be overwritten when the container is updated.**

## warriors

* URLTeam2
    * accessible on port 8000
    * runs 6 concurrent tasks³
* ArchiveTeam’s Choice
    * accessible on port 8001
    * runs 2 concurrent tasks

The downloader name for both is set to `warriorvm-dot-gecko-dot-space` by default, if you really care about it you can change this name by logging in via ssh, change directory to `~/warrior-vm`, editing `warrior.conf` and running `sudo docker-compose up -d`.

**Doing it this way will abort all your currently running archiving tasks!**

To prevent this you can press the `Shut down`-button on both warriors and wait until they are done with their tasks before changing the config.

In `warrior.conf` you can also set `HTTP_USERNAME` and `HTTP_PASSWORD` to enable HTTP Basic Auth for the web interfaces.

---

¹ for example "Deutsche Telekom" is well known to redirect non-existent domains to their own search engine; some countries mandate that ISPs have to block "bad domains" such as The Pirate Bay; some people use [Pi-Hole](https://pi-hole.net/) or [AdGuardHome](https://adguard.com/en/adguard-home/overview.html) to filter ads/trackers from their whole network

² you probably want to exclude this zvol from your periodic snapshot tasks

³ this sould be fine as they produce very little system load

# Huidu D15

Notes and stuff around bringing up a Huidu D15 board. Mostly as a learning experience for Yocto, but
also with the potential to be the basis of a pretty capable CNC control board.

I'm using Claude Schwarz'
[Twitter thread](https://twitter.com/Claude1079/status/1513541029399519234) as
reference/inspiration.

Claude Schwarz' Github repo is [here](https://github.com/captain-amygdala/huidu-hd-d16).

## References

- <https://wiki.t-firefly.com/en/Core-PX30-JD4/getting_started.html>
- <https://github.com/captain-amygdala/huidu-hd-d16>

## Serial

- "CoolTerm" seems to be an alright serial terminal for Windows.
  - Just use PuTTY
- 115200 baud works for me with default software, although
  [apparently](https://twitter.com/Claude1079/status/1512693171305779202) 1.5MBaud (1500000) is
  default for Rockchips.
- Connections:

  ![](images/serial.jpeg)

- U-Boot does indeed use 1.5MBaud

## Getting into the bootloader and changing the root password

- The serial console will start at 1.5Mbaud (default for Rockchip I think), then be reconfigured to
  115200 Baud.
- At this point, HOLD Ctrl + C as per the prompt, and now we have a uboot console.
  - Don't mash; the boot delay is zero, so mashing will likely not work but holding will.
- `env print` is useful
- A more complex solution for setting `init`:
  <https://community.toradex.com/t/passing-bootarg-init-bin-bash-to-u-boot/11141>
- However this seems to work fine, but the FS is readonly:

  ```
  => env edit bootargs
  (add the following to the end of the line:) init=/bin/sh
  => boot
  ```

- Running `mount` will give a `proc` not found error
  - Fix with `mount -t proc proc /proc`
  - Reference: https://unix.stackexchange.com/q/647916/41760
- Now remount `/` as writable: `mount -o remount,rw /`
- `passwd root`
- Power cycle the system

## Backing up the disk

- SSHd is disabled by default, but telnet isn't
- We need to sync the RTC, so ensure the battery is in the board and the board has internet access
  before boot - the NTP server will sync automatically. If we don't do this, you'll get a looping
  prompt to change your password when SSHing in.

  - Reference: <https://unix.stackexchange.com/questions/415757/password-expired-on-ssh>

- To run SSHd, generate some keys:

  - Reference: <https://michlstechblog.info/blog/linux-regenerate-sshd-host-keys/>
  - Note that some of the values change from the reference:
  - ```bash
    ssh-keygen -b 4096 -f /etc/ssh/ssh_host_rsa_key -t rsa -N ""
    ssh-keygen -b 1024 -f /etc/ssh/ssh_host_dsa_key -t dsa -N ""
    ssh-keygen -b 521 -f /etc/ssh/ssh_host_ecdsa_key -t ecdsa -N ""
    ssh-keygen -b 4096 -f /etc/ssh/ssh_host_ed25519_key -t ed25519 -N ""
    ```
  - Now run `/usr/bin/sshd`

- Remote `dd` reference: <https://unix.stackexchange.com/a/132800/41760>
- `ssh root@192.168.0.27 "dd if=/dev/mmcblk0 | gzip -1 -" | dd of=huidu-d15.gz`
- Or to a flash drive locally: `dd if=/dev/mmcblk0 bs=64 | gzip -1 > /root/usb_dev/huidu-d15.gz`
  - Very slow, CPU bound by `gzip`.

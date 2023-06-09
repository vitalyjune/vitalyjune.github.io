# explore the internals of qcow2 image under Windows by mounting it under Multipass virtual machine

I have qcow2 image created from my Linux machine, I downloaded it to my Windows machine as C:\Users\Vito\Downloads\ImageStorage\image.img and want to explore it.

1. Download Multipass for Windows from https://multipass.run/ and install it.

2. Run Multipass, choose "Open shell" and it will automatically download and install Ubuntu into its virtual machine.

3. shell: mkdir /home/ubuntu/my_windows_mount

4. Under Windows mounting local directories into Multipass Linux VM is disabled by default for security reasons (https://multipass.run/docs/privileged-mounts), we will turn it on.  
cmd: multipass set local.privileged-mounts=Yes

5. Mount folder with image into VM.  
cmd: multipass mount C:\Users\Vito\Downloads\ImageStorage primary:/home/ubuntu/my_windows_mount

6. check the folder is mounted, you should see the image file image.img  
shell: ls /home/ubuntu/my_windows_mount

next part is taken from https://docs.j7k6.org/mount-qcow2-disk-image-linux/

7. Install qemu-utils  
shell: apt install qemu-utils

8. Load nbd kernel module  
shell: modprobe nbd max_part=8

9. Mount QCOW2 disk image as NBD device  
shell: qemu-nbd -c /dev/nbd0 --read-only /home/ubuntu/my_windows_mount/image.img

10. List partitions on NBD device (optional)  
shell: fdisk -l /dev/nbd0

I have two partions inside qcow2 image - nbd0p1 (boot) and nbd0p2 (data), so fdisk output is:  

Disk /dev/nbd0: 100 GiB, 107374182400 bytes, 209715200 sectors  
Units: sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes 
I/O size (minimum/optimal): 512 bytes / 512 bytes  
Disklabel type: gpt  
Disk identifier: 123  

Device      Start       End   Sectors  Size Type  
/dev/nbd0p1  2048      4095      2048    1M BIOS boot  
/dev/nbd0p2  4096 209715166 209711071  100G Linux filesystem  

11. Mount NBD data partition’s filesystem  
shell: mount -o ro /dev/nbd0p2 /mnt  

**now cow2 image internals are read-only mounted in /mnt**

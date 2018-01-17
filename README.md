# yocto_build_steps
Yocto Build for ARM Development Board
# Yocto build for iMX6Q Technexion board

Pre-Build Yocto build

```
$:sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
 build-essential chrpath  socat \
 libsdl1.2-dev xterm  sed cvs subversion coreutils texi2html \
 docbook-utils python-pysqlite2 help2man make gcc g++ desktop-file-utils \
 libgl1-mesa-dev libglu1-mesa-dev mercurial autoconf automake groff curl lzop asciidoc
``` 
For Ubuntu 14.04 host setup only:

```
$: sudo apt-get install u-boot-tools
```
### Install Repo
```
$: mkdir ~/bin
$: curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$: chmod a+x ~/bin/repo
```
### Download the BSP source:
```
$: PATH=${PATH}:~/bin
$: mkdir edm_yocto
$: cd edm_yocto
$: repo init -u https://github.com/TechNexion/edm-yocto-bsp.git -b krogoth_4.1.y_GA
$: repo sync -j8
```
### TEP-1010-IMX6, QT5 with X11 image:

```
$: DISPLAY=lvds10v01 MACHINE=tek-imx6 source edm-setup-release.sh -b build-x11-tek -e x11
```
### Apply the patch provided from Technexion support team

```diff
bpc:~/edm_yocto/build-x11-tek/tmp/work/tek_imx6-poky-linux-gnueabi/u-boot-edm/2016.03-r0/git$ git diff board/technexion/tek-imx6/tek-imx6.c
diff --git a/board/technexion/tek-imx6/tek-imx6.c b/board/technexion/tek-imx6/tek-imx6.c
index ca4d800..e24aa3b 100755
--- a/board/technexion/tek-imx6/tek-imx6.c
+++ b/board/technexion/tek-imx6/tek-imx6.c
@@ -480,16 +480,16 @@ struct display_info_t const displays[] = {{
        .detect = NULL,
        .enable = enable_lvds,
        .mode   = {
-               .name           = "hj070na",
+               .name           = "10inch_v01",
                .refresh        = 60,
-               .xres           = 1024,
-               .yres           = 600,
-               .pixclock       = 15385,
-               .left_margin    = 220,
+               .xres           = 1280,
+               .yres           = 800,
+               .pixclock       = 1000000000000ULL/71100000,
+               .left_margin    = 40,
                .right_margin   = 40,
-               .upper_margin   = 21,
-               .lower_margin   = 7,
-               .hsync_len      = 60,
+               .upper_margin   = 10,
+               .lower_margin   = 3,
+               .hsync_len      = 50,
                .vsync_len      = 10,
                .sync           = FB_SYNC_EXT,
                .vmode          = FB_VMODE_NONINTERLACED
```
#### Bake the build
```
$: bitbake fsl-image-qt5
```
#### If changes made on kernel
```
$ bitbake -c clean fsl-image-qt5
$ bitbake fsl-image-qt5
```
#### After build sucessful
```
~/edm_yocto/build-x11-tek/tmp/deploy/images/tek-imx6$ ls
fsl-image-qt5-tek-imx6-20171229155638.rootfs.tar.bz2   README_-_DO_NOT_DELETE_FILES_IN_THIS_DIRECTORY.txt 
fsl-image-qt5-tek-imx6-20171230004250.rootfs.ext4      SPL                                                   zImage--4.1.15-2.0.0-r0-imx6q-tep5-20171229155638.dtb
fsl-image-qt5-tek-imx6-20171230004250.rootfs.manifest  SPL-tek-imx6                                          zImage--4.1.15-2.0.0-r0-tek-imx6-20171229155638.bin
fsl-image-qt5-tek-imx6-20171230004250.rootfs.sdcard    SPL-tek-imx6-2016.03-r0                               zImage-imx6dl-tek3.dtb
fsl-image-qt5-tek-imx6-20171230004250.rootfs.tar.bz2   u-boot.img                                            zImage-imx6dl-tep5.dtb
fsl-image-qt5-tek-imx6.ext4                            u-boot-tek-imx6-2016.03-r0.img                        zImage-imx6q-tek3.dtb
fsl-image-qt5-tek-imx6.manifest                        u-boot-tek-imx6.img                                   zImage-imx6q-tep5.dtb
fsl-image-qt5-tek-imx6.sdcard                          uEnv.txt                                              
zImage-tek-imx6.bin
```

### Flash the image .sdcard file to sdcard 
* Note please check correct drive lable for sdcard using `sudo fdisk -l` and check path like `/dev/sdb`

```
$ ~/edm_yocto/build-x11-tek/tmp/deploy/images/tek-imx6$ sudo dd if=fsl-image-qt5-tek-imx6-20171230004250.rootfs.sdcard of=/dev/sdb bs=1M && sync
```
## Edit the file for setting RGB colors in uEnv.txt file 
In my case `RGB666` did not correctly display proper colors on LVDS, to correct this change to `RGB22` 

### From 
```displayinfo=video=mxcfb0:dev=ldb,1280x800@60,if=RGB666
mmcargs=setenv bootargs console=${console},${baudrate} root=${mmcroot} ${displayinfo}
bootcmd_mmc=run loadimage;run mmcboot;
uenvcmd=run bootcmd_mmc
```

### To 
```
displayinfo=video=mxcfb0:dev=hdmi,1280x800@60,if=RGB24 fbmem=28M
mmcargs=setenv bootargs console=${console},${baudrate} root=${mmcroot} ${displayinfo}
bootcmd_mmc=run loadimage;run mmcboot;
uenvcmd=run bootcmd_mmc
```

Credits and for more support information refer [TechNexion](https://github.com/TechNexion/) page

# buildJetsonTX2Kernel
Scripts to help build the 4.4.38 kernel and modules onboard the Jetson TX2 (L4T 28.1, JetPack 3.1). For previous versions, visit the 'tags' section.

As of this writing, the "official" way to build the Jetson TX2 kernel is to use a cross compiler on a Linux PC. This is an alternative which builds the kernel onboard the Jetson itself. These scripts will download the kernel source to the Jetson TX2, wrangle some of the Makefiles to make them work on the Jetson, and then compile the kernel and selected modules. The newly compiled kernel can then be installed.

These scripts are for building the kernel for the 64-bit L4T 28.1 (Ubuntu 16.04 based) operating system on the NVIDIA Jetson TX2. The scripts should be run directly after flashing the Jetson with L4T 28.1 from a host PC. There are three scripts:

<strong>getKernelSources.sh</strong>

Downloads the kernel sources for L4T 28.1 from the NVIDIA website, decompresses them and opens a graphical editor on the .config file. In editor, select General Setup/Local Version - append to kernel release, and change name of the kernel to match you kenel name (e.g -jetsonTX). 

To add support of INZI pixel format for realsense SR300, follow the patch by realsense from https://www.spinics.net/lists/linux-media/msg108702.html and add following lines if not present:

usr/src/kernel/kernel-4.4/drivers/media/usb/uvc/uvcvideo.h in GUIDs macros:

    #define UVC_GUID_FORMAT_INVZ \
      { 'I',  'N',  'V',  'Z', 0x90, 0x2d, 0x58, 0x4a, \
       0x92, 0x0b, 0x77, 0x3f, 0x1f, 0x2c, 0x55, 0x6b}
    #define UVC_GUID_FORMAT_INZI \
      { 'I',  'N',  'Z',  'I', 0x66, 0x1a, 0x42, 0xa2, \
       0x90, 0x65, 0xd0, 0x18, 0x14, 0xa8, 0xef, 0x8a}
    #define UVC_GUID_FORMAT_INVI \
      { 'I',  'N',  'V',  'I', 0xdb, 0x57, 0x49, 0x5e, \
       0x8e, 0x3f, 0xf4, 0x79, 0x53, 0x2b, 0x94, 0x6f}


usr/src/kernel/kernel-4.4/drivers/media/usb/uvc/uvcvideo.h inside uvc_format_desc uvc_fmts[] struct:

	{
		.name		= "Depth data 16-bit (Z16)",
		.guid		= UVC_GUID_FORMAT_INVZ,
		.fcc		= V4L2_PIX_FMT_Z16,
	},
	{
		.name		= "IR:Depth 26-bit (INZI)",
		.guid		= UVC_GUID_FORMAT_INZI,
		.fcc		= V4L2_PIX_FMT_INZI,
	},
	{
		.name		= "Greyscale 10-bit (Y10 )",
		.guid		= UVC_GUID_FORMAT_INVI,
		.fcc		= V4L2_PIX_FMT_Y10,
	},


/usr/src/kernel/kernel-4.4/include/uapi/linux/videodev2.h inside Vendor-specific formats macros:

    #define V4L2_PIX_FMT_MT21C    v4l2_fourcc('M', 'T', '2', '1') /* Mediatek compressed block mode  */
    #define V4L2_PIX_FMT_INZI     v4l2_fourcc('I', 'N', 'Z', 'I') /* Intel Infrared 10-bit linked with Depth 16-bit */


<strong>makeKernel.sh</strong>

This script applies a few patches to makefiles in the kernel source, and then compiles the kernel and modules using make.

<strong>copyImage.sh</strong>

Copies the Image file created by compiling the kernel to the /boot directory

<strong>Notes:</strong> 

The kernel source files are downloaded in a .tgz2 format. After compilation you may want to remove those files. The files are located in /usr/src You will need to use sudo to remove the files, as they are in a system area. The work directory 'sources' contains kernel_src-txt2.tbz2, you can remove that directory. The file 'source_release.tbz2' is a much larger file that holds the kernel sources as well as many other TX2 specific source packages. You can make a backup of source_release.tbz2 before deleting it.

You may want to save the newly built Image and modules to external media so that can be used to flash a Jetson image, or clone the disk image.

The copyImage.sh script copies the Image to the current device. If you are building the kernel on an external device, for example a SSD, you will want to copy the Image file over to the eMMC in the eMMC /boot directory if you are booting from the eMMC and using external storage as your root directory. 





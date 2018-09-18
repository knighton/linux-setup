## Xubuntu 18.04 Deep Learning Computer Setup

Instructions and workarounds for setting up a computer with an NVIDIA GPU for deep learning, especially Alienware laptops.

### 1. Create installation media

Create a bootable Xubuntu 18.04 installation flash drive.

### 2. Boot from the installation media

Boot the machine into the Xubuntu flash drive.  This will probably require mucking with the BIOS settings: (a) disable SecureBoot, (b) disable FastBoot, (c) switch boot mode to Legacy then fix the boot order to try USB first.

### 3. Install to SSD/HDD

Do "Install Xubuntu".  "Try Xubuntu" won't work.  There is no try.  Do, or do not.

Some things may be too new for 18.04 driver support, but this can be fixed post install.  If the touchpad fails to work, plug in a mouse for now.  Etc.

Install with the "wipe everything and partition things automatically" option.  But it is certainly fine to configure it yourself if things are working.

### 4. Reboot and log in

Now reboot and you should be good to go.

* If Alienware touchpad fails to work, run

      $ su
      $ echo 'blacklist i2c_hid' >> /etc/modprobe.d/blacklist.conf
      $ depmod -a
      $ update-initramfs -u
      
* If you just get a black screen upon successful login, suspect you may have a graphics driver problem.  Kill nouveau and install the preferred graphics driver (in any case you'll need to switch graphics drivers anyway):

  Mount the installed OS filesystem, and edit the boot flags in /boot/grub/grub.cfg to add "nouveau.blacklist=1".  This probably goes inbetween "splash" and "$vt_handoff", but google this if in doubt.

  Then boot into the installed OS, and when facing the graphical login prompt, Ctrl+Alt+F-whatever into a shell, then do the next step to install a working driver.

### 5. Install graphics driver

First, create a root account (run "sudo passwd root" in a console).

Then install the NVIDIA driver.  If 390 is acceptable, you can do this and be done with it, because it doesn't automatically provide 396.

       $ ubuntu-drivers devices
       (should hopefully see your graphics card)

       $ ubuntu-drivers autoinstall
       (will take a while...)

       $ reboot

However, apparently you need 396 (the next major release) for pytorch.  For 396, simply run:

       $ add-apt-repository ppa:graphics-drivers/ppa
       $ apt update
       $ apt install nvidia-driver-396

If that fails because of an apt dependencies problem, aptitude works and will repair apt.

       $ apt install aptitude
       $ aptitude install nvidia-driver-396

### 6. Install CUDA

Install CUDA 9.0 (for TensorFlow) and then CUDA 9.2 (for PyTorch).

Downloads at https://developer.nvidia.com/cuda-download.

18.04 is not officially supported at this time.  Get the runfiles, not the .deb packages.  Select the options

       [Linux] [x86_64] [Ubuntu] [17.10] [runfile (local)]

Install CUDA libraries, but not the accompanying NVIDIA driver (*read prompts carefully*).  Install them in order.

       $ bash cuda_9.0.176_384.81_linux.run --override
       $ bash cuda_9.0.176.1_linux.run
       $ bash cuda_9.0.176.2_linux.run
       $ bash cuda_9.0.176.3_linux.run
       $ bash cuda_9.0.176.4_linux.run
 
Then

       $ bash cuda_9.2.148_396.37_linux.run --override
       $ bash cuda_9.2.148.1_linux.run

Then update your ~/.bashrc to add CUDA to your path:

       export PATH=/usr/local/cuda-9.0/bin:$PATH
       export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64:$LD_LIBRARY_PATH
       
       export PATH=/usr/local/cuda-9.2/bin:$PATH
       export LD_LIBRARY_PATH=/usr/local/cuda-9.2/lib64:$LD_LIBRARY_PATH

### 7. Install cuDNN

Install cuDNN 7.1.4 for CUDA 9.0 and 9.2.

Downloads at https://developer.nvidia.com/rdp/cudnn-download.

Unzip the files and put them where they belong:

       $ cp -ai include/* /usr/local/cuda-9.0/include/
       $ cp -ai lib64/* /usr/local/cuda-9.0/lib64/
    
And for the 9.2 versions:

       $ cp -ai include/* /usr/local/cuda-9.2/include/
       $ cp -ai lib64/* /usr/local/cuda-9.2/lib64/

### 9. Install tensorflow

It should just work.

       $ apt install python3-pip
       $ pip3 install tensorflow-gpu
 
### 10. Install pytorch

It should just work.

       $ pip3 install http://download.pytorch.org/whl/cu92/torch-0.4.1-cp36-cp36m-linux_x86_64.whl 
       $ pip3 install torchvision

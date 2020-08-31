# Posenet with Google Coral Dev Board

###### tags: `Posenet` `TPU` `Coral Dev Board`

> This readme divided into two parts.
> 1. How to set up Google Coral Dev Board?
> 2. How to set up environment for Posenet?

## :memo: How to set up Google Coral Dev Board?

### Part 1: install tools

Below is the link to google offical guide.
> https://coral.ai/docs/dev-board/get-started/#requirements

But, some of my steps are different. 
I will write my own steps to help you reduce your time solving errors. :smile: 

#### There are somethings you need to know first!!!
> Recommend to use sudo. If you don't use sudo, they will install in different folders. When you need to modify some default setting, there will be some errors.
> Always check the package you installed on board can be used in ARM64 system.


Below is my own steps.
#### Step 1: Install screen
Install screen to connect the coral dev board by using USB-A to USB-micro-B cable.
```
sudo apt-get install screen
```

#### Step 2: Install fastboot tools
Install fastboot tool to let you flash the board.

Below is the link to download fastboot tools from Android Studio.
>https://developer.android.com/studio/releases/platform-tools.

- We only need fastboot tools. Delete the rest of folders.

Then move fastboot tools into dev board.
```
mkdir -p ~/.local/bin
sudo mv ~/Downloads/platform-tools/fastboot ~/.local/bin/
```

Verify fastboot has been installed.
```
fastboot --version
```
- If you get the message like "the command was not found", please add fastboot into system command.

#### Step 3: Install MDT tools
- Before using pip3. Don't forget to install python and python-pip3. Please choose the right folder to install. Or you will have lots of errors.
```
pip3 install mendel-development-tool
```

### Part 2 : Connect dev board

Before connect to the dev board. Install udev.
- Linux
  Add udev rules by using below commands
```
sudo sh -c "echo 'SUBSYSTEM==\"usb\", ATTR{idVendor}==\"0525\", MODE=\"0664\", \
GROUP=\"plugdev\", TAG+=\"uaccess\"' >> /etc/udev/rules.d/65-edgetpu-board.rules"

sudo udevadm control --reload-rules && sudo udevadm trigger
```
##### If you can't echo the rules, try to download the rules by yourself. Then, put rules into that folders.

- Mac
  Download CP210x USB to UART Bridge Virtual COM Port (VCP) driver for Mac.
>https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers

    * Let the board in eMMC mode.
    * Connect USB-A to USB-micro-B cable first!
    * Connect the power cable

- On Linux
```
dmesg | grep ttyUSB

screen /dev/ttyUSB0 115200
```
##### Screen ttyUSB which showed in your terminal.
If there is nothing showed, try to check if USB function is be forwarded to your computer or not.

- On Mac
```
screen /dev/cu.SLAB_USBtoUART 115200
```

Fastboot the dev board.
```
fastboot 0
```

### Part 3 : Install system image
    * Unplug microUSB cable
    * Connect with USB-C cable
- Check if fastboot is done or not
```
fastboot devices
```
It may showed below message(number might be different)
```
1b0741d6f0609912        fastboot
```

#### If it's blank, there are 2 method to solve this error.
1. Make sure your fastboot tool has been installed secessfully.
2. If your fastboot tool has installed sucessfully and there is still blank, you can try to connect the board by using ssh.

- How to connect the board by using ssh?
Step 1. Unplug USB-C cable. Using microUSB cable to connect the dev board.
Step 2. There will be a login page.
Account : mendel Password : mendel
Step 3. Try to create the root user. Then relogin by using Account: Root.
Step 4. Try to add your computer's public ssh key into ./ssh file in coral dev board.
Step 5. Set the wireless network. Make sure that dev board and your computer are in the same network.
Step 6. Use ssh to connect Ip address which dev board connected to.

After you can connect to dev board sucessfully, we can start to install system image.
Below is the method by using ssh. If you want to use mdt tools, please follow google official guide.

- Use ssh connect your dev board.(Use your board IP address)
```
ssh -Y root@ 192.168.10.2
```
Download system image and install it.
```
cd ~/Downloads

curl -O https://mendel-linux.org/images/enterprise/eagle/enterprise-eagle-20200724205123.zip

unzip enterprise-eagle-20200724205123.zip \ 
&& cd enterprise-eagle-20200724205123

bash flash.sh
```

It may take 5-10 minutes to install. After installation, you finish setting up coral dev board. :+1: 

## :memo: How to set up environment for Posenet?

### Step 1 : Clone Posenet project

You have 2 options to clone Posenet project.
1. The original Posenet Github project.
```
git clone https://github.com/google-coral/project-posenet.git
```
2. The Github project modified by myself. I have modified some codes to avoid errors.
```
git clone --depth 1 https://github.com/Yun-Jung/project-posenet.git
```
#### In my environment, the following changes were necessary. Below are parts of codes which modified to work on dev board.
1. Add int for offset varibles.(Minor errors)
``` pose_engine.py=103
self._output_offsets.append(int(offset))
```
2. Change "waylandsink" and "autovideosink" to "fakesink" (This errors depend on own environment.)

In my environment, I can't use "waylandsink" or "autovideosink" to display the video from camera. To solve this issue, I modified the gstreamer.py.


### Step 2: Install the package for Posenet
```
./install_requirements.sh
```
```
sudo apt-get -y install curl apt-transport-https python-software-properties software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get -y install gcc-4.9
sudo apt-get -y upgrade libstdc++6
sudo apt-get -y install libtiff-dev libjpeg-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl-dev tk-dev python-tk
pip3 install Pillow
```
#### Install EdgeTpu to run Posenet on Coral Dev Board
```
echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install libedgetpu1-std
pip3 install https://dl.google.com/coral/python/tflite_runtime-2.1.0.post1-cp35-cp35m-linux_aarch64.whl
```
#### Install gstreamer for Posenet
```
sudo apt-get -y install python3-gi gobject-introspection gir1.2-gtk-3.0
```
### Step 3 : Test
#### After installing all of packages, you can try to run simple_pose.py to see if works or not.
```
python3 simple_pose.py
```
It will show x,y-coordinates of keypoints. If not, plese check your picture is jpg file. 
### Step 4 : Run Posenet by using camera
```
python3 pose_camera.py
```

There might be some problems when you run pose_camera.py. 
Below are errors I have met before.
1. Make sure your X-11 function is working.
2. If the error messages showed something about "waylandsink", please modify the codes in gstreamer.py. You can change both "waylandsink" and "autovideosink" into "fakesink".

## Part 3 : Things need to be done in the future
- [ ] Let posenet can use the camera which connected to coral dev board.
- [ ] Rewrite the gstreamer.py to let gstreamer run well on ARM64 system.
- [ ] Make good use of keypoints from Posenet.
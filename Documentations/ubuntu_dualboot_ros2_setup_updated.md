# Dual Booting Ubuntu 22.04 with Windows 10/11 & Installing ROS 2 Humble

This guide will walk you through:

1. Setting up a bootable USB to install Ubuntu 22.04 (Jammy Jellyfish)
2. Dual booting Ubuntu with Windows
3. Initial Ubuntu setup
4. Installing ROS 2 Humble Hawksbill on Ubuntu

---

## Step 1: Download Ubuntu 22.04 Desktop Image and BalenaEtcher

- **Ubuntu 22.04 ISO** (Desktop version):  
   [Download here](https://releases.ubuntu.com/jammy/)

- **Balena Etcher** (USB boot tool):  
   [Download here](https://etcher.balena.io/)

---

## Step 2: Create a Bootable USB Stick

1. Insert a **blank USB stick (8GB or more)** into your PC. **Note**: All data on it will be erased.
2. Open **Balena Etcher**:
   - **Flash from file** → Select the Ubuntu ISO you downloaded.
   - **Select target** → Automatically selected if one USB is connected. If more than one device is plugged in, select your USB manually.
   - **Click "Flash"** → Wait until the process completes.
3. Once done, **eject the USB safely**.

---

## Step 3: Prepare Windows for Dual Boot

### 3.1 Backup Your Important Data
Before modifying partitions or OS setups, **back up all important files** from your Windows system.

### 3.2 Shrink Your Windows Partition
1. Open **Disk Management**:
   - Press `Win + S` → Search for **"Create and format hard disk partitions"**.
2. Right-click the main Windows partition (usually `C:`) → **Shrink Volume**.
3. Enter the amount to shrink in **MB** (e.g., 50000 MB for 50GB).
4. Click **Shrink**. You'll see **"Unallocated space"** – this is where Ubuntu will be installed.

**⚠️ Important:**  
Do **not** touch:
- `EFI System Partition`
- `Recovery Partition`

---

## Step 4: Boot Into Ubuntu Installation

### 4.1 Enable USB Boot (if not already enabled)
- Restart the PC → Press `F2`, `DEL`, or `ESC` to access the BIOS/UEFI.
- Under **Boot Options**:
  - Enable **USB boot**.
  - Set USB as **first boot device**.
  - **Disable Secure Boot** (Recommended for smoother Ubuntu installs).

### 4.2 Boot from USB
1. Restart your PC.
2. Go to **Settings > System > Recovery > Advanced startup > Restart now**.
3. Choose **Use a device** → Select the USB.
4. A black screen with GRUB will appear → Choose **"Try or Install Ubuntu"**.

---

## Step 5: Install Ubuntu Alongside Windows

1. Choose your language.
2. Select **"Install Ubuntu"**.
3. Choose your **keyboard layout** (usually English – US, usually: qwerty).
4. For **Installation Type**:
   - Choose **Normal Installation**
   - Check:
     - ✅ "Download updates while installing Ubuntu"

5. Connect to **Wi-Fi** and click **Continue**.

### 5.1 Manual Partitioning ("Something Else")
1. From the partition table:
   - Find the **"free space"** you created earlier (be careful not to use the wrong one).
2. Click on the free space → **Add**:
   - Type: **Primary**
   - Location: **Beginning of this space**
   - Use as: **Ext4 journaling file system**
   - Mount point: `/`
3. Click OK.

### 5.2 Set Boot Loader Location
- Below the partition list, choose:
  - **Device for boot loader installation**: Select the entry labeled **Windows Boot Manager** (usually `/dev/sda1`).

6. Click **Install Now** → Confirm changes → Continue.

---

## Step 6: Finalize Installation

1. Choose your **timezone**.
2. Set:
   - Your **Name**
   - Your **Computer Name**
   - Your **Username**
   - Your **Password** (used for login and sudo commands)

3. Click **Continue** → Installation starts. It can take ~30–60 minutes.

---

## Step 7: Reboot & Access Ubuntu

1. Once installation completes, click **Restart Now**.
2. When prompted, **remove the USB stick** and press **Enter**.
3. On every startup, you’ll now see the **GRUB menu** to choose between:
   - Ubuntu
   - Windows Boot Manager

---

## Step 8: Post-Installation Checks

- Boot into both **Windows and Ubuntu** to confirm dual boot works.
- **Update Ubuntu**:
  ```bash
  sudo apt update && sudo apt upgrade
  ```

---

## Step 9: Install ROS 2 Humble on Ubuntu 22.04

## 9.1 System Setup

### Set Locale

Ensure your locale supports UTF-8. If you're using a minimal environment (like Docker), the default locale might be POSIX. To configure a UTF-8 compatible locale:

```bash
locale  # check current locale

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```

---

## 9.2 Add the ROS 2 APT Repository

### 9.2.1 Enable Ubuntu Universe Repository:

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
```

### 9.2.2 Add ROS 2 apt source:

Install the latest ROS 2 source package:

```bash
sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
```

---

## 9.3 Install Development Tools & ROS Tools

Install common development packages:

```bash
sudo apt update && sudo apt install -y \
  python3-flake8-docstrings \
  python3-pip \
  python3-pytest-cov \
  ros-dev-tools
```

Install additional tools for Ubuntu 22.04:

```bash
sudo apt install -y \
  python3-flake8-blind-except \
  python3-flake8-builtins \
  python3-flake8-class-newline \
  python3-flake8-comprehensions \
  python3-flake8-deprecated \
  python3-flake8-import-order \
  python3-flake8-quotes \
  python3-pytest-repeat \
  python3-pytest-rerunfailures
```

---

## 9.4 Get ROS 2 Code

Create a workspace and clone the source code:

```bash
mkdir -p ~/ros2_humble/src
cd ~/ros2_humble
vcs import --input https://raw.githubusercontent.com/ros2/ros2/humble/ros2.repos src
```

---

## 9.5 Install Dependencies using `rosdep`

Ensure your system is updated, then install dependencies:

```bash
sudo apt upgrade
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"
```

>  If you're using a derivative like Linux Mint, and get an error such as `Unsupported OS [mint]`, append:
> ```bash
> --os=ubuntu:jammy
> ```

---

## 9.6 Optional: Install Additional DDS Implementations

If you'd like to use DDS or RTPS vendors other than the default, refer to:

[DDS Vendor Installation Guide](https://docs.ros.org/en/humble/Installation/Alternatives/DDS-Implementations.html)

## Official ROS 2 Humble Installation Guide

For more details, visit the official ROS 2 documentation:  
[https://docs.ros.org/en/humble/Installation.html](https://docs.ros.org/en/humble/Installation.html)
 
---

## Optional Troubleshooting: GRUB Menu Not Showing

Sometimes after Ubuntu installation, the system might boot straight into Windows, skipping the GRUB menu. To fix this:

### Step 1: Make GRUB the Default Bootloader
Boot into **Ubuntu** (you might need to use the Boot Menu key like `F12` or `ESC` during startup).

Open a terminal and run:
```bash
sudo grub-install
sudo update-grub
```

This ensures GRUB is properly installed and configured.

---

### Step 2: Change UEFI Boot Order to Boot Ubuntu First
1. Reboot your computer and enter the BIOS/UEFI settings (`F2`, `DEL`, `ESC`, or as per your device).
2. Look for **Boot Order** or **Boot Priority** settings.
3. Move **Ubuntu** or `grubx64.efi` (usually under `/EFI/ubuntu/`) **above** the Windows Boot Manager.
4. Save and exit.

This ensures GRUB loads before Windows Boot Manager.

---

### Step 3: If Windows Keeps Overriding GRUB
From **Windows**, open **Command Prompt as Administrator** and run:
```cmd
bcdedit /set {bootmgr} path \EFI\ubuntu\grubx64.efi
```

This tells the Windows Boot Manager to defer to GRUB.

---

### Step 4: Add Windows Entry to GRUB (If Missing)
Back in Ubuntu, run:
```bash
sudo os-prober
sudo update-grub
```

This scans for Windows and adds it to the GRUB boot menu.

---

### ✅ Final Result:
On reboot, you should now see a menu like:
```
Ubuntu
Advanced options for Ubuntu
Windows Boot Manager (on /dev/nvme0n1p1)
```




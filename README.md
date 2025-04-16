### MacOS4Boxes

This Is a how to for running Mac OS on Gnome Boxes.

This document Is based on the `README.md` from https://github.com/kholia/OSX-KVM. For more details, please, go to kholia's OSX-KVM github.

### Requirements

* A modern Linux distribution. E.g. Debian 12 64-bit or later (I used Debian 13 for my setup).

* QEMU >= 8.2.2

* A CPU with Intel VT-x / AMD SVM support is required (`grep -e vmx -e svm /proc/cpuinfo`)

* A CPU with SSE4.1 support is required for >= macOS Sierra

* A CPU with AVX2 support is required for >= macOS Ventura

### Installation Preparation

* Install QEMU and other packages.

  ```
  sudo apt install qemu-system uml-utilities gnome-boxes git wget libguestfs-tools p7zip-full make dmg2img tesseract-ocr tesseract-ocr-eng genisoimage vim net-tools screen -y
  ```

  This step may need to be adapted for your Linux distribution.

* Clone this repository on your QEMU system. Files from this repository are
  used in the following steps.

  ```
  cd ~

  git clone --depth 1 --recursive https://github.com/kholia/OSX-KVM.git

  cd OSX-KVM
  ```

* KVM may need the following tweak on the host machine to work.

  ```
  sudo modprobe kvm; echo 1 | sudo tee /sys/module/kvm/parameters/ignore_msrs
  ```

  To make this change permanent, you may use the following command.

  ```
  sudo cp kvm.conf /etc/modprobe.d/kvm.conf  # for intel boxes only

  sudo cp kvm_amd.conf /etc/modprobe.d/kvm.conf  # for amd boxes only
  ```

* Add user to the `kvm` and `libvirt` groups (might be needed).

  ```
  sudo usermod -aG kvm $(whoami)
  sudo usermod -aG libvirt $(whoami)
  sudo usermod -aG input $(whoami)
  ```

  Note: Re-login after executing this command.

* Fetch macOS installer.

  ```
  ./fetch-macOS-v2.py
  ```

  You can choose your desired macOS version here. After executing this step,
  you should have the `BaseSystem.dmg` file in the current folder.

  ATTENTION: Let `>= Big Sur` setup sit at the `Country Selection` screen, and
  other similar places for a while if things are being slow. The initial macOS
  setup wizard will eventually succeed.

  Sample run:

  ```
  $ ./fetch-macOS-v2.py
  1. High Sierra (10.13)
  2. Mojave (10.14)
  3. Catalina (10.15)
  4. Big Sur (11.7)
  5. Monterey (12.6)
  6. Ventura (13) - RECOMMENDED
  7. Sonoma (14)
  8. Sequoia (15)

  Choose a product to download (1-8): 6
  ```

  Note: Modern NVIDIA GPUs are supported on HighSierra but not on later
  versions of macOS.

* Convert the downloaded `BaseSystem.dmg` file into the `BaseSystem.img` file.

  ```
  dmg2img -i BaseSystem.dmg BaseSystem.img
  ```

* Create a virtual HDD image where macOS will be installed. If you change the
  name of the disk image from `mac_hdd_ng.img` to something else, the boot scripts
  will need to be updated to point to the new image name.

  ```
  qemu-img create -f qcow2 mac_hdd_ng.img 256G
  ```

  NOTE: Create this HDD image file on a fast SSD/NVMe disk for best results.

* Edit `macOS-libvirt-Catalina.xml` file and change the various file paths (search
    for `CHANGEME` strings in that file). The following command should do the
    trick usually.

    ```
    sed "s/CHANGEME/$USER/g" macOS-libvirt-Catalina.xml > macos.xml

    virt-xml-validate macos.xml
    ```
* Edit the file `/etc/qemu/bridge.conf`, if the file don't exist, create It.
    ```
    sudo mkdir /etc/qemu
    sudo vim /etc/qemu/bridge.conf
    ```
* Add the line `allow virbr0` or `allow all` and change the file's permission.
    ```
    sudo chmod u+r /etc/qemu/bridge.conf
    ```
* Add `setuid` to the `qemu-bridge-helper` binary.
    ```
    sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper
    ```
* Start the `default` network bridge, and configure it to run on startup.
    ```
    sudo virsh net-autostart --network default
    sudo virsh net-start --network default
    ```

* Confirm If the `virbr0` Is up.
    ```
    ip addr show virbr0
    ```

* Import the `xml` file to work with the Gnome Boxes.
    ```
    virsh create macos.xml
    ```

* Now you are ready to install macOS on `Gnome Boxes`.

### Sources
Below are the links I use to create this document.

https://github.com/kholia/OSX-KVM and https://mike42.me/blog/2019-08-how-to-use-the-qemu-bridge-helper-on-debian-10

Feel free to point any errors or updates on this document or copy this document.

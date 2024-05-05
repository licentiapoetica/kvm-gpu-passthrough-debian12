# GPU Passthrough on Debian 12 for kvm

This document works for me and was tested against my current setup.

```
       _,met$$$$$gg.           
    ,g$$$$$$$$$$$$$$$P.       -------- 
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 12 (bookworm) x86_64 
 ,$$P'              `$$$.     Host: X570 Taichi 
',$$P       ,ggs.     `$$b:   Kernel: 6.1.0-20-amd64 
`d$$'     ,$P"'   .    $$$    Uptime: 8 hours, 15 mins 
 $$P      d$'     ,    $$P    Packages: 3498 (dpkg) 
 $$:      $$.   -    ,d$$'    Shell: zsh 5.9 
 $$;      Y$b._   _,d$P'      Resolution: 1920x1080, 1920x1080, 1920x1080 
 Y$$.    `.`"Y$$$$P"'         DE: Plasma 5.27.5 
 `$$b      "-.__              WM: KWin 
  `Y$$                        Theme: [Plasma], Breeze [GTK2/3] 
   `Y$$.                      Icons: breeze-dark [Plasma], breeze-dark [GTK2/3] 
     `$$b.                    Terminal: konsole 
       `Y$$b.                 CPU: AMD Ryzen 7 5800X (16) @ 3.800GHz 
          `"Y$b._             GPU: AMD ATI Radeon RX 470/480/570/570X/580/580X/590 
              `"""            GPU: AMD ATI Radeon RX 6600/6600 XT/6600M 
```

## What you most likely need in order for it to work pretty much as you expect it

At least 2 dedicated GPUs (In my case both my GPUs are AMD cards)

A Board that puts devices in their own IOMMU groups and not into a big one!!!

Optional but useful: A board like the Taichi X570 that has the right pci-e slots for the operation, meaning that your second slot is capable of at least pci-e 3.0 x8 otherwise you will most likely encounter poor performance in video games and trust me I have been there with a B450 board.

## Resources and verifying your hardware

For the most part you should consult https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF even if you are not on Debian.

Now for the Debian specific stuff...

First of all make sure that IOMMU is enabled and working, you can check this by running

```
# dmesg | grep -i -e DMAR -e IOMMU
```

Should report back a lot of stuff but at the bottom it should say something like this:

`[1.031747] AMD-Vi: AMD IOMMUv2 loaded and initialized`

See Arch wiki Article 2.2 for a script that lists your IOMMU groups, verify that your secondary GPU is in it's own group.

For example take a look at my output:

```
IOMMU Group 30:
        10:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X/590] [1002:67df] (rev e1)
        10:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere HDMI Audio [Radeon RX 470/480 / 570/580/590] [1002:aaf0]
```

You can see that my secondary GPU is in it's own isolated IOMMU Group, this is what you need.

Now do this

```
sudo vim etc/modprobe.d/vfio.conf 

## add the following to the file
options vfio-pci ids=1002:67df,1002:aaf0

--

echo 'vfio-pci' > /etc/modules-load.d/vfio-pci.conf 

sudo vim /etc/initramfs-tools/modules

## add the following to the file
vfio_pci
vfio
vfio_iommu_type1

sudo update-initramfs -u
```

Now reboot

Check `sudo dmesg | grep -i vfio`

If the output contains

```
[    6.647395] VFIO - User Level meta-driver version: 0.3
[    6.649940] vfio_pci: add [1002:67df[ffffffff:ffffffff]] class 0x000000/00000000
[    6.650246] vfio_pci: add [1002:aaf0[ffffffff:ffffffff]] class 0x000000/00000000
```

it worked and you can now use the GPU in a virtual machine for example.

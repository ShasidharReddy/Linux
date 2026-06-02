# Device Drivers

This guide covers Linux driver models, device discovery, sysfs, udev, and driver lifecycle basics.

Device drivers are the kernel components that mediate between generic kernel subsystems and hardware-specific behavior.

## 11.1 Driver Categories

| Category | Description |
|---|---|
| Character driver | Stream or byte-oriented interface |
| Block driver | Block device interface for storage |
| Network driver | Packet-oriented NIC integration |
| Platform driver | Non-discoverable platform devices |
| Bus driver | USB, PCI, I2C, SPI, etc. |

## 11.2 Character vs Block Devices

**Character devices** are generally accessed as streams or arbitrary byte operations.

Examples:

- Serial ports
- `/dev/null`
- TTY devices

**Block devices** support block-oriented random access.

Examples:

- Disks
- SSDs
- Loop devices

## 11.3 Device Nodes in `/dev`

Device files under `/dev` provide user-space handles to devices.

Each has:

- Major number identifying driver class
- Minor number identifying device instance/subtype

Inspect:

```bash
ls -l /dev/null
lsblk
```

## 11.4 `udev`

`udev` is the user-space device manager that reacts to kernel uevents and populates `/dev` with device nodes and symlinks.

## 11.5 sysfs

`sysfs` exposes kernel object relationships and device model state under `/sys`.

Examples:

- `/sys/class/net/`
- `/sys/block/`
- `/sys/devices/`
- `/sys/bus/pci/devices/`

## 11.6 Device Model Concepts

Linux device model includes:

- Devices
- Drivers
- Buses
- Classes
- kobjects
- sysfs representation

## 11.7 Device Trees

On many embedded systems, hardware description uses **device tree** data structures to describe buses, devices, and configuration.

## 11.8 PCI and Discovery

On PCs and servers, buses like PCIe support discovery. Kernel enumerates devices and matches them to drivers via IDs.

## 11.9 Driver Binding

A driver binds to a device when supported identifiers or compatible strings match and initialization succeeds.

## 11.10 Common Driver Interfaces

Drivers frequently implement callbacks through subsystem operation tables.

Examples:

- `file_operations` for char devices
- `net_device_ops` for network devices
- block layer request hooks

## 11.11 Mermaid-Free Mental Model for Drivers

Think of a driver as a translator between:

- Generic kernel subsystem contracts
- Register-level or protocol-level device behavior

## 11.12 Minimal Character Device Driver Skeleton

```c
#include <linux/fs.h>
#include <linux/module.h>
#include <linux/uaccess.h>

static ssize_t demo_read(struct file *f, char __user *buf, size_t len, loff_t *off)
{
    return 0;
}

static const struct file_operations demo_fops = {
    .owner = THIS_MODULE,
    .read = demo_read,
};

static int major;

static int __init demo_init(void)
{
    major = register_chrdev(0, "demochr", &demo_fops);
    return major < 0 ? major : 0;
}

static void __exit demo_exit(void)
{
    unregister_chrdev(major, "demochr");
}

module_init(demo_init);
module_exit(demo_exit);
MODULE_LICENSE("GPL");
```

## 11.13 Interrupt Handling in Drivers

Drivers often register interrupt handlers.

Common pattern:

- Top half acknowledges urgent device event
- Deferred work handles heavier processing later

## 11.14 DMA

Devices often transfer data using **Direct Memory Access**.

Drivers must:

- Allocate suitable buffers
- Map/unmap memory for DMA
- Respect device addressing constraints
- Handle cache coherency rules on relevant architectures

## 11.15 Power Management

Modern drivers participate in:

- Suspend/resume
- Runtime power management
- Clock and regulator coordination on embedded platforms

## 11.16 Module Writing Basics

Typical workflow:

1. Write kernel module source.
2. Build against matching kernel headers.
3. Load with `insmod` or `modprobe`.
4. Inspect logs with `dmesg`.
5. Unload with `rmmod` if safe.

## 11.17 Example Makefile for Module

```make
obj-m += demo.o
KDIR ?= /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
```

## 11.18 Useful Commands for Driver Work

```bash
lspci -nnk
lsusb
udevadm info -a -p /sys/class/net/eth0
modinfo e1000e
dmesg | tail -100
```

## 11.19 Common Driver Failure Modes

| Symptom | Cause |
|---|---|
| Device not appearing | Enumeration or binding failure |
| Spurious interrupts | Misconfigured IRQ handling |
| DMA corruption | Mapping/coherency bug |
| Kernel oops on unload | Lifetime/reference error |
| Suspend resume issues | Power management callback bug |

## 11.20 Section Summary

Drivers are where Linux meets hardware reality. Even if you never write drivers, understanding their structure helps explain performance anomalies, device failures, and the meaning of `/dev`, `/sys`, and kernel logs.

---

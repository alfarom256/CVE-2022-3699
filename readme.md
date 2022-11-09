## CVE-2022-3699

Incorrect access control for the Lenovo Diagnostics Driver allows a low-privileged user the ability to issue device IOCTLs to perform arbitrary physical/virtual memory read/write.

Thank you to ch3rn0byl for helping with this (and I totally 100% ripped two of his functions).

### Explanation

IOCTL 0x222000:

* rdmsr

IOCTL 0x222008/0x22200C:

* HalGet/SetBusData

IOCTL 0x222010:

* Read via MmMapIoSpace

IOCTL 0x222014:

* Write via MmMapIoSpace

* This IOCTL copies a value from a pointer supplied in the input buffer into mapped physical memory


### How it works:

In order to resolve MmPteBase and other prerequisites, a physical "swap" space is found by searching the physical memory range `0x1000 - 0x10000` for 8 zero bytes.

Once that space is found, virtual memory is copied into that swap space via IOCTL 0x222014 and read back using IOCTL 0x222010.

As it is now, ALL virtual reads are done using this "swap" space. 

Is it the best way to do virtual r/w? Probably not.

Does it work? Yes.

Oh, also, mind your own offsets -- this was tested on Windows 11 21H2 with HVCI disabled.
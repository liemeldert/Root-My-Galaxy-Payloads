# S21+ US Snapdragon unlocked port: SM-G996U1-G996U1UESJHZB1

I used samloader-rs, as the others, to obtain the firmware update files for the latest release.

`G996WVLSJHZAA/G996WOYVJHZAA/G996WVLSJHZAA/G996WVLSJHZAA`

From there, I extracted the zip and the AP. To get the boot.img and kernel, I used this modified version of the script in porting.md just to make it easier to verify I was getting correct results.

```py
from pathlib import Path
import hashlib
import struct

boot = Path("boot.img").read_bytes()

magic = boot[0x00:0x08]
kernel_size = struct.unpack_from("<I", boot, 0x08)[0]
ramdisk_size = struct.unpack_from("<I", boot, 0x0C)[0]
header_version = struct.unpack_from("<I", boot, 0x28)[0]

print("boot magic ok:", magic == b"ANDROID!", magic)
print("header version:", header_version)
print("kernel_size:", kernel_size)
print("ramdisk_size:", ramdisk_size)
print("boot.img size:", len(boot))
print("boot.img sha256:", hashlib.sha256(boot).hexdigest().upper())

kernel = boot[0x1000:0x1000 + kernel_size]
Path("kernel").write_bytes(kernel)

arm_magic = struct.unpack_from("<I", kernel, 0x38)[0]
text_offset = struct.unpack_from("<Q", kernel, 0x08)[0]
image_size = struct.unpack_from("<Q", kernel, 0x10)[0]
flags = struct.unpack_from("<Q", kernel, 0x18)[0]

print("arm64 magic ok:", arm_magic == 0x644D5241, hex(arm_magic))
print("text_offset:", hex(text_offset))
print("image size:", hex(image_size))
print("flags:", hex(flags))
print("kernel size:", len(kernel))
print("kernel sha256:", hashlib.sha256(kernel).hexdigest().upper())
```

The result of which being the following:

```
boot magic ok: True b'ANDROID!'
header version: 3
kernel_size: 52451840
ramdisk_size: 723407
boot.img size: 100663296
boot.img sha256: 15BB692E944A3F6D02DEBD5C69835AE790D333ABC4AFC42DA43C8D54B54A7D2F
arm64 magic ok: True 0x644d5241
text_offset: 0x80000
image size: 0x37a5000
flags: 0xa
kernel size: 52451840
kernel sha256: 8463C2E2380333B2374B6BF34B3C13CFBC66E5A8FCA6429F35DCEC36D9B0E6D9
```



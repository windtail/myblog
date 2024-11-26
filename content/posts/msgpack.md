---
title: "Msgpack"
date: 2024-11-26T17:48:02+08:00
draft: false
tags:
  - python
  - msgpack
---

claude 总结的msgpack，写得挺好的！


I'll help explain how MessagePack handles integer and double (floating point) number representations.

```python
import struct
import binascii

def demonstrate_msgpack_numbers():
    # Integer representations
    def pack_fixnum(n):  # -32 to 127
        return bytes([n & 0xff]) if n >= 0 else bytes([n & 0xff])
    
    def pack_uint8(n):  # 0 to 255
        return bytes([0xcc, n & 0xff])
        
    def pack_uint16(n):  # 0 to 65535
        return bytes([0xcd]) + struct.pack(">H", n)
        
    def pack_uint32(n):  # 0 to 2^32-1
        return bytes([0xce]) + struct.pack(">I", n)
        
    def pack_uint64(n):  # 0 to 2^64-1
        return bytes([0xcf]) + struct.pack(">Q", n)
        
    def pack_int8(n):  # -128 to 127
        return bytes([0xd0]) + struct.pack("b", n)
        
    def pack_int16(n):  # -32768 to 32767
        return bytes([0xd1]) + struct.pack(">h", n)
        
    def pack_int32(n):  # -2^31 to 2^31-1
        return bytes([0xd2]) + struct.pack(">i", n)
        
    def pack_int64(n):  # -2^63 to 2^63-1
        return bytes([0xd3]) + struct.pack(">q", n)
    
    # Float representations
    def pack_float32(f):
        return bytes([0xca]) + struct.pack(">f", f)
        
    def pack_float64(f):
        return bytes([0xcb]) + struct.pack(">d", f)

    # Examples
    examples = [
        # Integers
        ("Positive fixnum (7)", pack_fixnum(7)),
        ("Negative fixnum (-16)", pack_fixnum(-16)),
        ("uint8 (200)", pack_uint8(200)),
        ("uint16 (65000)", pack_uint16(65000)),
        ("uint32 (70000)", pack_uint32(70000)),
        ("uint64 (2**40)", pack_uint64(2**40)),
        ("int8 (-100)", pack_int8(-100)),
        ("int16 (-1000)", pack_int16(-1000)),
        ("int32 (-70000)", pack_int32(-70000)),
        ("int64 (-2**40)", pack_int64(-2**40)),
        
        # Floats
        ("float32 (3.14159)", pack_float32(3.14159)),
        ("float64 (3.14159265359)", pack_float64(3.14159265359))
    ]
    
    # Print results
    print("MessagePack Number Representations:")
    print("==================================")
    for desc, packed in examples:
        print(f"\n{desc}:")
        print(f"Hex: {binascii.hexlify(packed).decode()}")
        print(f"Raw bytes: {' '.join(f'{b:02x}' for b in packed)}")

# Run the demonstration
demonstrate_msgpack_numbers()

```

MessagePack uses different formats to represent integers and floating-point numbers efficiently. Here's how they work:

Integers:
1. Fixnum (single byte):
   - Positive: 0x00 to 0x7f (0 to 127)
   - Negative: 0xe0 to 0xff (-32 to -1)

2. Unsigned integers:
   - uint8: 0xcc followed by 1 byte (0 to 255)
   - uint16: 0xcd followed by 2 bytes (0 to 65535)
   - uint32: 0xce followed by 4 bytes (0 to 2^32-1)
   - uint64: 0xcf followed by 8 bytes (0 to 2^64-1)

3. Signed integers:
   - int8: 0xd0 followed by 1 byte (-128 to 127)
   - int16: 0xd1 followed by 2 bytes (-32768 to 32767)
   - int32: 0xd2 followed by 4 bytes (-2^31 to 2^31-1)
   - int64: 0xd3 followed by 8 bytes (-2^63 to 2^63-1)

Floating-point numbers:
1. float32 (single precision):
   - Format: 0xca followed by 4 bytes in IEEE 754 format
   - Uses big-endian byte order

2. float64 (double precision):
   - Format: 0xcb followed by 8 bytes in IEEE 754 format
   - Uses big-endian byte order

The code artifact above provides a complete demonstration of how these numbers are encoded, with examples of each type. You can run it to see the exact byte representation of different numbers in MessagePack format.

Would you like me to explain any specific aspect of the encoding in more detail?
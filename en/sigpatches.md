# Preparing a sigpatched SKSA

### Prerequisites

- Familiarity with commandline tools
- A hex editor (I recommend HxD on Windows and also HxD on Linux via wine)
- iQueCrypt and familiarity with its usage, or another way to encrypt/decrypt data with AES-CBC-128; pycryptodomex works for this
- iQueTool and familiarity with its usage, or another way to build NAND images with arbitrary SKSAs
- A copy of SKSA 1106 (other SKSAs will work, but you'll have to find the SA2 patch locations yourself)
- The common key, SK key and SK IV, or a copy of the bootrom and a Virage2 dump in order to extract them

### Extracting the common key, SK key and SK IV from a copy of the bootrom and a Virage2 dump

The common key is 16 bytes long and is located at offset 0xB8 in Virage2. Its SHA-1 hash is `42B344BA419F44FC0A021E86375464CE8A130935`.

The SK key is 176 bytes long and is located at offset 0x13C0 in the bootrom. You only need the last 16 bytes of that, starting at 0x1460 in the bootrom. The SHA-1 hash of the bit you need is `B5D47F1770D4EAB391D6294885B273E7013F9257`.

The SK IV is 16 bytes long and is located at offset 0x1470 in the bootrom. Its SHA-1 hash is `2BC7A0F488D83F98E94B42C619DBE42B7AAF0BA5`.

### Contents

[Extracting everything](#extracting-everything)

[Patching](#patching)

[Rebuilding everything](#rebuilding-everything)

[What next?](#what-next)

### Extracting everything

You may want to use marshallh's [ique_sksa_decrypt](https://github.com/marshallh/ique_sksa_decrypt) in order to do the decryption.

Alternatively:

#### Decrypting SK

SK takes up the first 0x10000 bytes (64KiB) of the SKSA blob. Copy it out to a file.

Then, use iQueCrypt's `decrypt` mode with the `-app` flag to decrypt it, using the SK key and IV.

#### Decrypting SA1

You don't need to decrypt SA1. It doesn't need to be patched.

#### Decrypting SA2

The layout of the SKSA blob is:

- SK (0..0x10000)
- SA1 ticket block (0x10000..0x14000)
- SA1 (0x14000..0x14000 + SA1 size)
- SA2 ticket block (0x14000 + SA1 size..0x18000 + SA1 size)
- SA2 (0x18000 + SA1 size..0x18000 + SA1 size + SA2 size)

You can determine SA1 and SA2's sizes by reading the SA1 and SA2 CMD heads, located at the start of their respective ticket blocks. Alternatively, 1106's SA2 ticket block starts at 0x30000 and its SA2 starts at 0x34000. Copy it out to a file.

Grab the key IV ("commonCmdIv") and encrypted key ("key") from SA2's CMD head. Use iQueCrypt to decrypt the key, using the common key as the kek (key encryption key).

Then, grab the content IV ("iv") from SA2's CMD head. Use iQueCrypt to decrypt SA2, using the decrypted key you just got and the content IV. Check the hash of the decrypted blob against the hash stored in SA2's CMD head.

#### Decompressing SA2

Decompress the decrypted SA2 blob. `gzip` won't work for this, since it's headerless.

Here's a nice one-liner to do it if you have Python 3 installed:
```bash
$ cat sa2-compressed.bin | python3 -c "import zlib;import sys;sys.stdout.buffer.write(zlib.decompress(sys.stdin.buffer.read(), -15))" > sa2.bin
```

The equivalent command in PowerShell:
```powershell
gc sa2-compressed.bin -AsByteStream | py -3 -c "import zlib;import sys;sys.stdout.buffer.write(zlib.decompress(sys.stdin.buffer.read(), -15))" > sa2.bin
```

### Patching

There are 4 patches to make to SK:

- 0x0C10 (`9FC00C10`): `14620054` → `14630054`
- 0x0FEC (`9FC00FEC`): `2645FFDC` → `27A50010`
- 0x161C (`9FC0161C`): `02002821` → `00042821`
- 0x33E4 (`9FC033E4`): `24A50004` → `24850000`

There is one patch to make to SA2:

- 0x1C248 (`80432418`): `27A50304` → `27A50100`

### Rebuilding everything

There is a tool to rebuild an SKSA blob from component parts, but it won't generate correct blobs for this.

#### Encrypting SK

Use iQueCrypt's `encrypt` mode with the `-app` flag to encrypt the modified SK, using the SK key and IV.

The hash, once encrypted, should be `509D22ADD440FEEA2DF00BE6506A7DBB02D36A4E`.

#### Encrypting SA1

You don't need to encrypt SA1, since it doesn't need to be patched.

#### Compressing SA2

Use `gzip` to compress the data with compression level 9, and then remove the header. A one-liner to do this using `dd`:
```bash
$ gzip -c -9 -n sa2-dec.bin | dd bs=10 skip=1 of=sa2-compressed.bin
```

#### Encrypting SA2

Pad the encrypted SA2 binary to the next multiple of 0x4000 bytes (16KiB). This should be 0xB8000.

Calculate the SHA-1 hash of the padded binary and save it for later. It should be `DBD40D8737CC8C2DE625B07DF7FD5ADD9276C25F`.

Encrypt the padded binary using iQueCrypt, using the decrypted key and content IV from earlier. The hash of the encrypted binary should be `1AE5CCFF09261AEE4B3D11DFF955DBFB28EDDCD5`.

#### Putting it together

Overwrite the first 0x10000 bytes of your SKSA blob with your encrypted, modified SK.

Overwrite the blob's SA2 section (0x34000..end in 1106) with your encrypted, compressed, modified SA2.

Overwrite the hash in SA2's CMD head (0x30024 in 1106) with the hash of the unencrypted, padded, compressed, modified SA2 from earlier (`DBD40D8737CC8C2DE625B07DF7FD5ADD9276C25F`).

Save the modified blob somewhere.

### What next?

Congratulations, you're done! The hash of your final modified SKSA should be `A56B637F1BACB5EA4B989B80966487FDB5D1326F`. Now you can write it to your console to try it out!

Make sure your console is fully jailbroken before you do this, since otherwise it won't boot, and you'll need to borrow someone else's card to recover it.

You can use iQueTool to update the SKSA in a NAND dump, and then write it to your console using iQueDiagExtend's `2` command (use `2 0-0x40` to only overwrite the SKSA area, saving a huge amount of time).

Here's the command to generate a new NAND and spare:
```bash
$ iquetool nand -uk modified.sksa -gs spare.bin -o nand.bin old-nand.bin
```

Enjoy!

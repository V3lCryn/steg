# steg

A binary analysis and steganography toolkit for CTF challenges, written in [Volta](https://github.com/V3lCryn/volta) — a custom programming language that compiles to C99.

```
[=] FILE INFO
    Path    : challenge.png
    Size    : 48291 bytes
    Type    : png
    Entropy : 7.81 (sample 4096 bytes)

[=] FLAG SCAN
    [+] +12440 (0x30a8)  picoCTF{h1dd3n_1n_pl41n_s1ght}
    [*] 1 match(es)

[=] FILE CARVING  (embedded file signatures)
    [ZIP]   at +47103 (0xb7df)
    [*] 1 signature(s) found

[=] APPENDED DATA DETECTION
    PNG IEND at +47088 (0xb7d0)
    [!] 1203 bytes of appended data at +47100
```

---

## What it does

`steg` runs a suite of forensics analysis modules against any binary file. Useful for CTF image steg challenges, binary analysis, and file format inspection.

| Flag | What it does |
|---|---|
| `--info` | File type detection, size, djb2 hash, entropy summary |
| `--header` | Hex dump of the first 64 bytes with ASCII sidebar |
| `--flags` | Scans for CTF flag patterns (picoCTF, HTB, THM, CTF, FLAG, etc.) |
| `--strings` | Extracts printable string runs of 6+ characters |
| `--entropy` | Per-256-byte-block Shannon entropy heatmap |
| `--carve` | Scans for embedded file signatures (JPEG, PNG, GIF, PDF, ZIP, RAR, GZIP, ELF, BMP) |
| `--lsb` | Extracts the LSB bit-stream from each byte and reassembles it |
| `--xor` | Brute-forces all 255 single-byte XOR keys, scores by printability, previews best result |
| `--append` | Detects data appended after JPEG EOI or PNG IEND markers |
| `--chunks` | Lists JPEG segment markers or PNG chunk structure, flagging non-standard chunks |
| `--freq` | Byte frequency histogram over the first 4096 bytes |
| `--fix` | Attempts to repair a corrupted JPEG or PNG file header |
| `--all` | Runs every module |

Default (no flags): runs `--info`, `--flags`, and `--carve`.

---

## Usage

```bash
# Compile with the Volta compiler
volta build steg.vlt -o steg

# Basic scan (info + flags + carve)
steg challenge.png

# Full analysis
steg challenge.png --all

# Targeted modules
steg mystery.jpg --header --append --chunks
steg encoded.bin --xor
steg image.png --lsb --entropy

# Repair a corrupted header
steg broken.jpg --fix
```

---

## Example outputs

**Entropy heatmap — spotting encrypted regions:**
```
[=] ENTROPY  (256-byte blocks — high = encrypted/compressed/random)
    offset   entropy  histogram
    +0       3.21     |||
    +256     7.94     ||||||||  << high (encrypted/compressed?)
    +512     7.88     ||||||||  << high (encrypted/compressed?)
    +768     1.02     |         << low (sparse/padding)
```

**XOR brute force:**
```
[=] XOR BRUTE-FORCE  (all single-byte keys 0x01–0xFF)
    Best key : 0x3f  (201/256 printable)
    [!] FLAG with key 0x3f: CTF{x0r_1s_t00_eas y}
    Decrypted preview (key 0x3f):
    CTF{x0r_1s_t00_easy} padding padding padding...
```

**PNG chunk inspection:**
```
[=] STRUCTURE / CHUNK ANALYSIS
    PNG chunks:
    offset    type  length  note
    +8        IHDR  len=13
    +33       IDAT  len=41032
    +41077    tEXt  len=512   << NON-STANDARD (possible hidden data!)
    +41601    IEND  len=0
```

---

## File types supported

Detection and analysis support for: JPEG, PNG, GIF, BMP, PDF, ZIP, RAR, GZIP, ELF, EXE.

Chunk/segment structure analysis: JPEG, PNG.

Header repair: JPEG, PNG.

---

## Why Volta

`steg` is written entirely in [Volta](https://github.com/V3lCryn/volta), a custom language with Lua-like syntax that compiles to C99. It uses Volta's C interop (`@extern "C"`) to call `fopen`/`fread`/`fwrite` directly for true binary file I/O — necessary because text-mode I/O would corrupt null bytes in binary data.

This makes `steg` a real-world demonstration of Volta's C interop, pointer handling, and low-level binary manipulation capabilities.

---

## Building

You need the Volta compiler. Clone it and follow the build instructions:

```
https://github.com/V3lCryn/volta
```

Then:

```bash
volta build steg.vlt -o steg
./steg <file> [options]
```

---

## Background

Built for CTF forensics work. Covers the most common image steg techniques encountered in competitions: LSB encoding, appended data, embedded files, XOR obfuscation, non-standard chunks, and corrupted headers.

---

## Licence

MIT

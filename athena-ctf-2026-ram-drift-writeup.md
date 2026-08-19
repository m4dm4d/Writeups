# Athena CTF (2026) — "RAM Drift" Writeup

**Category:** Forensics (RAM / Memory Forensics)
**Difficulty:** Medium
**Flag format:** `athena{...}`

---

## Introduction

This is a challenge I solved during Athena CTF (2026) that I found particularly interesting. RAM forensics isn't one of my strongest areas, but I decided to give it a shot — and ended up solving it.

## Challenge Overview

In this challenge, named **RAM DRIFT**, players are handed two plain-text artifacts recovered from a simulated crash-dump collector:

1. `ram_dump.txt`
2. `process_map.txt`

The goal is to reassemble the fragmented bytes hidden across these two files. At first glance, it seems like the flag might be sitting in one of the files in plain sight — just heavily obfuscated.

> 📷 *`process_map.txt` — screenshot placeholder*
>
> 📷 *`ram_dump.txt` — screenshot placeholder*

## My Workflow During the CTF

### Step 1 — Initial Attempt

My first instinct was to take the XOR tag from `process_map.txt` and XOR it directly with the bytes of the virtual address of the given memory page. That didn't produce anything meaningful — the result was still obfuscated.

### Step 2 — Spotting the Pattern

I took a closer look at the memory page `0x8a0000`. The `process_map.txt` file listed an XOR tag at that virtual address: `DR1FT`.

XOR-ing that tag against the first few bytes of the memory dump at `0x8a0000` produced a string starting with `athen...` — which closely resembles the flag format `athena{...}`. That was the confirmation I needed: the tag `DR1FT` was the XOR key.

**Important:** convert the string tag `DR1FT` to bytes before XOR-ing it against anything.

```python
>>> s = b"DR1FT"
>>> c = s[0]
>>> print(chr(c ^ 0x25))
a
```

### Step 3 — Decoding Byte by Byte

Continuing with the next few bytes:

```python
>>> s = b"DR1FT"
>>> print(chr(s[0] ^ 0x25))
a
>>> print(chr(s[1] ^ 0x26))
t
>>> print(chr(s[2] ^ 0x59))
h
>>> print(chr(s[3] ^ 0x23))
e
>>> print(chr(s[4] ^ 0x3a))
n
>>> print(chr(s[5] ^ 0x25))
Traceback (most recent call last):
  File "<python-input-17>", line 1, in <module>
    print(chr(s[5] ^ 0x25))
          ~~^^^
IndexError: index out of range
```

Since the key `DR1FT` is only 5 bytes long, trying to access `s[5]` throws an `IndexError`. This confirms the key must be **reused cyclically** (repeating XOR key) across the rest of the dump to decode the full flag.

### Step 4 — Handling Garbage Values

Repeating the 5-byte tag and XOR-ing it against `ram_dump.txt` line by line mostly works — but there's a catch. Looking at the memory page starting at `0x8a0000`, in the next offset line (`0000010`), there's a chunk that spells out `deadbeef` — a deliberately inserted garbage value. This needs to be **skipped/omitted** during the XOR process.

> 📷 *Screenshot showing `deadbeef` garbage bytes — placeholder*

The same thing happens again a bit further along, where a chunk spells out `faceb00c`. This also needs to be skipped.

> 📷 *Screenshot showing `faceb00c` garbage bytes — placeholder*

### Step 5 — Automating It

Once you know to:
1. Repeat the 5-byte key `DR1FT` cyclically, and
2. Skip over the `deadbeef` and `faceb00c` garbage blocks,

...it's much easier to just write a small Python script that reads `ram_dump.txt` line by line, skips the garbage markers, and XORs the remaining bytes against the repeating key — continuing until you hit the closing `}` of the flag.

## Result

After XOR-decoding the full dump (minus the garbage blocks), the flag comes out to:

```
athena{ram_pages_hide_fragments}
```

## Summary

This challenge was a nice exercise in pattern recognition within memory forensics — spotting that a short tag in a process map doubles as a repeating XOR key, and learning to filter out intentionally planted garbage values (`deadbeef`, `faceb00c`) that are common "noise" markers in memory dumps. A fun one for anyone looking to get more comfortable with RAM forensics challenges.

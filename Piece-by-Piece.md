# picoCTF 2026 — Piece by Piece

**Category:** General Skills  
**Difficulty:** Easy  
**Points:** 50

## Challenge Description

After logging in to the provided Linux environment, the flag is split across multiple files.

We are told to combine the pieces into a ZIP archive and extract it.

## Initial Enumeration

After connecting over SSH:

```bash
ssh -p <PORT> ctf-player@<HOST>
```

List the directory:

```bash
ls
```

The important files are:

```text
instructions.txt
part_aa
part_ab
part_ac
part_ad
part_ae
```

Read the instructions:

```bash
cat instructions.txt
```

The instructions tell us that:

- the flag is split into multiple file parts;
- the parts form a ZIP archive;
- the ZIP password is:

```text
supersecret
```

## Reconstructing the Archive

Because the parts are named sequentially:

```text
part_aa
part_ab
part_ac
part_ad
part_ae
```

the shell glob can concatenate them in lexical order:

```bash
cat part_* > part.zip
```

Check the resulting file:

```bash
file part.zip
```

It should be identified as a ZIP archive.

## Extracting the Flag

Use the supplied password:

```bash
unzip part.zip
```

When prompted:

```text
password: supersecret
```

A file named:

```text
flag.txt
```

is extracted.

Read it:

```bash
cat flag.txt
```

## Why It Works

The challenge is mainly about recognizing that the `part_*` files are fragments of one larger binary file.

Concatenating them reconstructs the original archive:

```text
part_aa + part_ab + part_ac + part_ad + part_ae
                         ↓
                       part.zip
                         ↓
                    unzip archive
                         ↓
                      flag.txt
```

## Key Takeaways

- Shell globbing can be used to concatenate ordered file fragments.
- `file` is useful for identifying reconstructed binary formats.
- Always read challenge instructions before trying more complicated approaches.

## Flag

```text
picoCTF{REDACTED}
```

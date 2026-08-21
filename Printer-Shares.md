# picoCTF 2026 — Printer Shares

**Category:** General Skills  
**Difficulty:** Easy  
**Points:** 50

## Challenge Description

The challenge hints that an important file was accidentally sent to a network printer.

The important clue is **printer shares**, which points toward SMB.

## Initial Enumeration

First confirm that the provided port is reachable:

```bash
nc -vz <HOST> <PORT>
```

A successful connection tells us that something is listening on the challenge port.

Since the challenge references shares, try SMB enumeration.

## Enumerating SMB Shares

Use `smbclient` with anonymous access:

```bash
smbclient -N -L //<HOST>/ -p <PORT>
```

The `-L` option lists the available shares.

A share named:

```text
shares
```

is exposed to guests.

## Accessing the Share

Connect to the share:

```bash
smbclient -N //<HOST>/shares -p <PORT>
```

Inside `smbclient`, list the files:

```text
ls
```

The important file is:

```text
flag.txt
```

## Downloading the Flag

From `smbclient`:

```text
get flag.txt
```

Then locally:

```bash
cat flag.txt
```

You can also use the command directly:

```bash
smbclient -N //<HOST>/shares -p <PORT> -c 'get flag.txt'
cat flag.txt
```

## Why It Works

The challenge is a simple information-disclosure/misconfiguration problem.

The server exposes an SMB share that allows guest access, and the sensitive file is readable through that share.

The key is recognizing the protocol from the challenge wording rather than treating the port as a generic TCP service.

## Key Takeaways

- `smbclient` is a useful tool for SMB enumeration.
- Always try anonymous/guest access when a CTF explicitly suggests a public share.
- Enumerate shares before guessing share names.
- Misconfigured file shares can expose sensitive information without requiring code execution.

## Flag

```text
picoCTF{REDACTED}
```

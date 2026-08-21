# picoCTF 2026 — bytemancy 0

**Category:** General Skills  
**Difficulty:** Easy  
**Points:** 50

## Challenge Description

The challenge asks us to connect to a remote service and provide the correct bytes.

The service tells us:

```text
Send me ASCII DECIMAL 101, 101, 101, side-by-side, no space.
```

## Analysis

The important part is the ASCII value:

```text
101
```

In ASCII:

```text
101 (decimal) = 0x65 = 'e'
```

The challenge wants the character represented by decimal `101`, repeated three times.

Therefore the required input is:

```text
eee
```

## Connecting to the Service

Use the `nc` command supplied by the challenge:

```bash
nc <HOST> <PORT>
```

Then enter:

```text
eee
```

A one-liner also works:

```bash
printf 'eee\n' | nc <HOST> <PORT>
```

## Why It Works

The program compares the input against three copies of the byte `0x65`:

```python
if user_input == "\x65\x65\x65":
    ...
```

Since `0x65` is the ASCII character `e`, the expected input is simply:

```text
eee
```

## Key Takeaways

- Decimal ASCII values can be converted directly into characters.
- `0x65` is hexadecimal for decimal `101`.
- Small encoding/conversion problems are often solved faster by checking the source code first.

## Flag

```text
picoCTF{REDACTED}
```

# picoCTF 2026 — bytemancy 1

**Category:** General Skills  
**Difficulty:** Easy  
**Points:** 100

## Challenge Description

The service asks for:

```text
ASCII DECIMAL 101 1751 times, side-by-side, no space.
```

## Analysis

We already know:

```text
101 (decimal) = 0x65 = 'e'
```

The important difference from `bytemancy 0` is the repetition count:

```text
1751
```

So the service expects:

```text
e
```

repeated exactly `1751` times.

Typing that manually is unnecessary and error-prone, so the hint about using Python is useful.

## Generating the Payload

The easiest solution is:

```bash
python3 -c 'print("e"*1751)'
```

This creates a string containing exactly 1751 `e` characters.

We can send it directly to the remote service:

```bash
python3 -c 'print("e"*1751)' | nc <HOST> <PORT>
```

## Why It Works

The supplied source checks the input against:

```python
"\x65" * 1751
```

Since:

```text
\x65 = e
```

the condition is effectively:

```python
user_input == "e" * 1751
```

## Key Takeaways

- Convert byte values to their character representation before doing anything complicated.
- Let Python generate large repetitive payloads.
- Piping generated data directly into `nc` avoids copy/paste mistakes.

## Flag

```text
picoCTF{REDACTED}
```

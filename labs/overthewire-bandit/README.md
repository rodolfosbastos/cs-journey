# OverTheWire — Bandit

Documenting my progress and learnings through the Bandit wargame.
Bandit focuses on Linux fundamentals — essential foundation for
any cybersecurity professional.

> ⚠️ This document tracks concepts learned, not solutions or passwords.

---

## Progress

| Level | Status | Core Concept |
|-------|--------|--------------|
| 0     | ✅ | SSH basics — connecting to a remote host |
| 0→1   | ✅ | Reading files with `cat` |
| 1→2   | ✅ | Reading files with special characters in their name |
| 2→3   | ✅ | Handling spaces in filenames |
| 3→4   | ✅ | Hidden files (dotfiles) |
| 4→5   | ✅ | Identifying file types with `file` |
| 5→6   | ✅ | Finding files by multiple properties with `find` |
| 6→7   | ✅ | Searching the whole filesystem by owner and size |
| 7→8   | ✅ | Searching inside files with `grep` |
| 8→9   | ✅ | Finding unique lines with `sort` and `uniq` |
| 9→10  | ✅ | Extracting readable strings from binary files |
| 10→11 | ✅ | Decoding Base64 |
| 11→12 | ✅ | ROT13 cipher and the `tr` command |
| 12→13 | ✅ | Hexdumps, file signatures, and decompression chaining |
| 13→14 | ✅ | SSH authentication with a private key |
| 14→15 | ✅ | Sending data to a port with `telnet` |
| 15→16 | ✅ | SSL/TLS encrypted connections with `openssl s_client` |
| 16→17 | ✅ | Port scanning with `nmap`, SSL service discovery, RSA private key retrieval |
| 17→25 | ⏳ | Pending |

---

## Learnings by Level

### Level 0 — SSH Basics
Connecting to a remote server via SSH for the first time. SSH (Secure Shell) is the standard protocol for secure remote access. You specify a host, a user, and in Bandit's case a non-default port.

```bash
ssh -p <port> <user>@<host>
```

**Key idea:** `-p` overrides the default port 22. Most real-world servers use non-standard ports as a minor hardening measure.

---

### Level 0→1 — Reading Files
The simplest operation: reading the contents of a file.

```bash
cat <filename>
```

**Key idea:** `cat` (concatenate) prints file contents to stdout. The starting point of almost every Linux workflow.

---

### Level 1→2 — Files with Special Names
Some filenames start with `-`, which the shell interprets as a flag/option rather than a filename.

```bash
cat ./-        # prefix with ./ to force path interpretation
cat < -        # use stdin redirection as alternative
```

**Key idea:** The shell parses `-` as "read from stdin." Prefixing with `./` tells it explicitly this is a file path, not an option.

---

### Level 2→3 — Spaces in Filenames
Filenames with spaces need to be escaped or quoted so the shell doesn't split them into separate arguments.

```bash
cat "spaces in this filename"
cat spaces\ in\ this\ filename
```

**Key idea:** The shell tokenizes on whitespace by default. Quotes or backslash escaping preserve the filename as a single argument.

---

### Level 3→4 — Hidden Files (Dotfiles)
Files prefixed with `.` are hidden from a standard `ls` listing. The `-a` flag reveals them.

```bash
ls -a
ls -la        # also shows permissions and ownership
```

**Key idea:** Dotfiles are a Unix convention for configuration and hidden data — not a security control, just convention. Always check with `-a` when something seems missing.

---

### Level 4→5 — Identifying File Types
When you have many files and need to find the human-readable one, `file` inspects content rather than relying on extensions.

```bash
file ./*       # run file against everything in current directory
```

**Key idea:** File extensions mean nothing to Linux — the OS uses magic bytes (the first bytes of a file) to determine type. `file` reads those magic bytes.

---

### Level 5→6 — Finding Files by Properties
`find` is one of the most powerful Linux commands. You can filter by size, permissions, type, ownership, and more — and combine filters.

```bash
find . -type f -size <n>c ! -executable
```

**Key idea:** Stacking `find` predicates with `!` (NOT), `-and`, `-or` lets you express precise queries. This mirrors how analysts hunt for anomalous files on a system.

---

### Level 6→7 — Searching the Whole Filesystem
Expanding the search scope to `/` (the entire filesystem) while filtering by owner user, owner group, and size. Errors from unreadable directories can be suppressed.

```bash
find / -user <user> -group <group> -size 33c 2>/dev/null
```

**Key idea:** `2>/dev/null` discards stderr (permission denied errors), keeping output clean. On a real system, searching by owner is a useful way to find files planted by a specific account.

---

### Level 7→8 — Searching Inside Files
When the password is stored in a large file next to a specific keyword, `grep` finds the right line instantly.

```bash
grep "keyword" data.txt
```

**Key idea:** `grep` is indispensable for log analysis, incident response, and reconnaissance. It supports regex, recursive search (`-r`), and can invert matches (`-v`).

---

### Level 8→9 — Finding the Unique Line
A file full of duplicate lines, with one line appearing only once. `sort` orders the lines, `uniq -u` prints only non-repeated ones.

```bash
sort data.txt | uniq -u
```

**Key idea:** Piping — chaining commands where the output of one becomes the input of the next — is fundamental to Unix. `uniq` only works on adjacent duplicates, so `sort` first is required.

---

### Level 9→10 — Strings in Binary Files
Binary files contain non-printable bytes, but may embed readable ASCII text. `strings` extracts sequences of printable characters.

```bash
strings <file>             # extract readable strings
strings <file> | grep "pattern"   # filter for relevant output
```

**Key idea:** `strings` is a basic but real forensics tool. Analysts use it to extract artifacts (URLs, credentials, error messages) from binaries and memory dumps.

---

### Level 10→11 — Base64 Encoding
The file contents are Base64 encoded. Base64 is a way to represent binary data as ASCII text — commonly used in emails, JWTs, and data embedded in web pages.

```bash
base64 -d data.txt
```

**Key idea:** Base64 is encoding, not encryption. It offers no confidentiality. Recognise it by its character set (`A-Z`, `a-z`, `0-9`, `+`, `/`) and `=` padding at the end.

---

### Level 11→12 — ROT13 / Caesar Cipher
The data is shifted 13 positions through the alphabet — ROT13. The `tr` command translates characters according to a mapping.

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**Key idea:** ROT13 is a trivial substitution cipher — applying it twice returns the original. Caesar ciphers are the simplest classical ciphers and a foundation for understanding modern encryption concepts.

---

### Level 12→13 — Hexdumps and Decompression Chaining
The file is a hexdump of data that has been compressed multiple times using different algorithms. You must reverse the hexdump, identify each layer by its file signature (magic bytes), and decompress repeatedly.

```bash
xxd -r hexdump > file       # reverse hexdump to binary
file <file>                  # check what type it is
# then decompress with gzip, bzip2, or tar as needed — repeat
```

**Key idea:** Files can be wrapped in multiple compression layers. `file` reads magic bytes to identify each layer regardless of extension. This is a practical introduction to file forensics and the importance of understanding file formats at the byte level.

---

### Level 13→14 — SSH Key Authentication + SCP
Instead of a password, this level provides a private SSH key file. Rather than using it directly on the server, the key file was downloaded to the local machine first using `scp`, then used to authenticate as the next user.

```bash
scp -P <port> bandit13@<host>:~/sshkey.private .    # download key to local machine
ssh -i sshkey.private bandit14@<host> -p <port>      # authenticate with the key
```

**Key idea:** SSH supports two authentication modes — password and public key. Key-based auth is more secure: the server stores only the public key, and the private key stays on your machine. `scp` (Secure Copy) uses the SSH protocol to transfer files between machines — same security, different purpose.

---

### Level 15→16 — SSL/TLS Encrypted Connections
Like level 14→15, but the service requires an encrypted connection. `openssl s_client` acts as a general-purpose SSL/TLS client — like telnet but for encrypted services.

```bash
openssl s_client -connect <host>:<port> -quiet   # connect with SSL, suppress TLS noise
```

**Key idea:** `-quiet` suppresses TLS session noise and disables interactive command mode (which interprets keystrokes like `k`/`K` as TLS commands rather than data). Without it, characters in your input can trigger unintended TLS handshake actions like KEYUPDATE. Always use `-quiet` when piping data to an SSL service non-interactively.

---

### Level 16→17 — Port Scanning and SSL Service Discovery
The correct port is unknown — one of several in a range speaks SSL and returns a private key. `nmap` scans the range to identify open ports and their services.

```bash
nmap -sV -p <start>-<end> localhost    # scan port range, detect service versions
openssl s_client -connect localhost:<port> -quiet  # connect and submit password
```

The server returns an **RSA private key** instead of a plain-text password — it must be saved and used with SSH `-i` to log in as the next user (same as level 13→14).

**Key idea:** `nmap -sV` probes each open port and fingerprints what service is running (e.g. `ssl/unknown` vs `ssl/echo`). Service version scanning is a core recon technique. An "echo" service just mirrors your input back — immediately ruled out as a candidate.

---

### Level 14→15 — Sending Data Over a Port
A service is running on the local machine listening on port 30000. To get the next password, you connect to that port and send the current level's password as plain text — the service validates it and responds with the next one.

```bash
telnet localhost 30000   # connect to the local service
# then type the password and hit enter
```

**Key idea:** Telnet establishes a raw text connection between two endpoints — an IP and a port. The process is identical whether the host is localhost or a remote machine. This makes telnet useful for testing and debugging services: you can talk directly to anything that communicates in plain text, seeing exactly what the protocol looks like without a client hiding it from you.

---

## Command Reference

```bash
# SSH
ssh -p <port> <user>@<host>          # connect with custom port
ssh -i <keyfile> <user>@<host>       # connect with private key

# File reading
cat <file>                            # print file contents
cat ./-                               # read file named with a dash
strings <file>                        # extract readable text from binary

# File discovery
ls -la                                # list all files including hidden
file ./*                              # identify types of all files
find . -size <n>c                     # find by exact size (c = bytes)
find / -user <u> -group <g> 2>/dev/null  # find by owner, suppress errors

# Text processing
grep "word" file                      # search for a word in a file
sort file | uniq -u                   # find unique (non-duplicate) lines

# Encoding / decoding
base64 -d file                        # decode Base64
cat file | tr 'A-Za-z' 'N-ZA-Mn-za-m'  # ROT13
xxd -r hexdump > binary               # reverse hexdump to binary

# Network communication
telnet <host> <port>                  # open raw text connection to a host:port
openssl s_client -connect <host>:<port> -quiet  # SSL/TLS encrypted connection
nmap -sV -p <start>-<end> <host>     # scan port range and detect service versions

# Decompression
gzip -d file.gz
bzip2 -d file.bz2
tar -xf file.tar
```

---

*Last updated: March 2026*

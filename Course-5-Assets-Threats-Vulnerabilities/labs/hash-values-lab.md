# Create Hash Values — File Integrity Verification Lab

## Project Description

In this project, I used Linux command-line tools to generate and compare SHA-256 cryptographic hash values to verify file integrity. Using the `sha256sum` and `cmp` commands, I demonstrated how two files that appear visually identical can contain hidden differences detectable only through hash comparison. This exercise directly mirrors the file integrity verification techniques used daily by SOC analysts to identify tampered files, detect malware, and validate the authenticity of digital evidence during security investigations.

---

## Scenario

Two files — `file1.txt` and `file2.txt` — appeared to contain identical content when viewed with the `cat` command. Both displayed the EICAR Standard Antivirus Test File string, a commonly used test signature in security environments. The task was to determine whether the files were truly identical or contained hidden differences by generating and comparing their SHA-256 hash values.

---

## Task 1 — Generate Hash Values

### Step 1 — List directory contents

```bash
ls
```

```
Output:
file1.txt  file2.txt
```

Confirmed both files exist in the current working directory `/home/analyst`.

---

### Step 2 — Display file contents

```bash
cat file1.txt
```

```
Output:
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

```bash
cat file2.txt
```

```
Output:
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

Both files appeared visually identical when displayed with `cat`. However, visual inspection alone is not a reliable method for verifying file integrity hidden characters, metadata differences, or subtle modifications are invisible to the human eye but detectable through hashing.

---

### Step 3 — Generate SHA-256 hashes

```bash
sha256sum file1.txt
```

```
Output:
131f95c51cc819465fa1797f6ccacf9d494aaaff46fa3eac73ae63ffbdfd8267  file1.txt
```

```bash
sha256sum file2.txt
```

```
Output:
2558ba9a4cad1e69804ce03aa2a029526179a91a5e38cb723320e83af9ca017b  file2.txt
```

Despite the files appearing visually identical, the SHA-256 hashes are completely different confirming that the files contain hidden differences that `cat` could not reveal. This demonstrates the power of cryptographic hashing for detecting even the smallest modifications to file content.

---

## Task 2 — Compare Hash Values

### Step 4 — Write hashes to separate files

```bash
sha256sum file1.txt >> file1hash
sha256sum file2.txt >> file2hash
```

The `>>` operator appends the hash output to new files — `file1hash` and `file2hash` — for comparison and documentation purposes.

---

### Step 5 — Display and compare hash files

```bash
cat file1hash
```

```
Output:
131f95c51cc819465fa1797f6ccacf9d494aaaff46fa3eac73ae63ffbdfd8267  file1.txt
```

```bash
cat file2hash
```

```
Output:
2558ba9a4cad1e69804ce03aa2a029526179a91a5e38cb723320e83af9ca017b  file2.txt
```

---

### Step 6 — Compare files byte by byte using cmp

```bash
cmp file1hash file2hash
```

```
Output:
file1hash file2hash differ: char 1, line 1
```

The `cmp` command confirmed that the hash files differ starting at the very first character of the first line proving that `file1.txt` and `file2.txt` contain different content despite their identical visual appearance.

---

## Complete Command Summary

```bash
# Navigate to working directory
pwd
# Output: /home/analyst

# List files
ls
# Output: file1.txt  file2.txt

# Display file contents
cat file1.txt
cat file2.txt

# Generate SHA-256 hashes
sha256sum file1.txt
sha256sum file2.txt

# Write hashes to separate files
sha256sum file1.txt >> file1hash
sha256sum file2.txt >> file2hash

# Display hash files
cat file1hash
cat file2hash

# Compare hashes byte by byte
cmp file1hash file2hash
```

---

## Hash Comparison Results

| File | SHA-256 Hash | Identical? |
|------|-------------|------------|
| file1.txt | 131f95c51cc819465fa1797f6ccacf9d494aaaff46fa3eac73ae63ffbdfd8267 | ❌ No |
| file2.txt | 2558ba9a4cad1e69804ce03aa2a029526179a91a5e38cb723320e83af9ca017b | ❌ No |

```
Visual inspection (cat):  Files appear IDENTICAL ❌ misleading
SHA-256 hash comparison:  Files are DIFFERENT    ✅ accurate
cmp command result:        Differ at char 1 line 1
```

---

## Key Findings

### Finding 1 — Visual inspection is unreliable
```
Both files displayed identical content
when viewed with the cat command

This demonstrates why visual inspection
alone is NEVER sufficient for:
→ Malware detection
→ File integrity verification
→ Digital forensics evidence validation
→ Software authenticity confirmation
```

### Finding 2 — SHA-256 reveals hidden differences
```
The completely different hash values prove
that file1.txt and file2.txt contain
differences invisible to human inspection

Real world implication:
→ A malicious file could be disguised
  to look identical to a legitimate file
→ Only hash comparison reveals the truth
→ One changed byte = completely different hash
  (avalanche effect of SHA-256)
```

### Finding 3 — cmp provides byte-level precision
```
The cmp command identified the difference
at character 1, line 1 — the earliest
possible position in the file

This byte-level comparison provides:
→ Exact location of tampering
→ Forensic precision for investigations
→ Irrefutable evidence of modification
```

---

## Real World SOC Analyst Applications

### Malware Detection
```bash
# Analyst receives suspicious executable
sha256sum suspicious_file.exe
# Output: d4e5f6a7b8c9...

# Compare against known malware database (VirusTotal)
# Hash match = confirmed malicious ⚠️
# No match = new or unknown malware variant
```

### File Integrity Monitoring
```bash
# Generate baseline hashes of critical system files
sha256sum /etc/passwd >> baseline_hashes.txt
sha256sum /etc/shadow >> baseline_hashes.txt

# Compare daily against baseline
sha256sum /etc/passwd | cmp - baseline_hashes.txt
# Difference detected = system file tampered ⚠️
```

### Digital Forensics Evidence Validation
```bash
# Generate hash of forensic evidence image
sha256sum evidence_drive.img >> evidence.hash

# Verify hash before and after analysis
# Hash must match to prove chain of custody
# Any difference = evidence compromised
```

### Software Authenticity Verification
```bash
# Download software package
sha256sum downloaded_software.tar.gz

# Compare against vendor published hash
# Match = authentic download ✅
# Mismatch = corrupted or tampered download ❌
```

---

## Commands Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `sha256sum file` | Generate SHA-256 hash of a file | `sha256sum file1.txt` |
| `sha256sum file >> hashfile` | Save hash to a file | `sha256sum file1.txt >> file1hash` |
| `cat hashfile` | Display saved hash value | `cat file1hash` |
| `cmp file1 file2` | Compare two files byte by byte | `cmp file1hash file2hash` |
| `ls` | List directory contents | `ls` |
| `pwd` | Show current directory | `pwd` |

---

## Why SHA-256 is the Gold Standard

```
MD5 (outdated):
→ 128-bit output
→ Collisions have been found
→ Two different files can produce
  the same MD5 hash
→ NOT suitable for security use ❌

SHA-1 (deprecated):
→ 160-bit output
→ Theoretical collisions demonstrated
→ No longer recommended ❌

SHA-256 (current standard):
→ 256-bit output
→ 2^256 possible hash values
→ No practical collisions possible
→ Used by governments, banks,
  militaries, and Bitcoin ✅
→ The right tool for file integrity ✅
```

---

## NIST CSF Alignment

```
IDENTIFY:
→ Identified file integrity as a
  critical security verification need
→ Established hash baseline for files

PROTECT:
→ Implemented cryptographic controls
  to verify data integrity
→ SHA-256 protects against undetected
  file tampering and malware

DETECT:
→ Hash comparison detects modifications
  that visual inspection misses
→ cmp command identifies exact location
  of tampering for investigation

RESPOND:
→ Hash evidence used in incident response
→ Confirms whether files were tampered
  during a security investigation
```

---

## Summary

This lab demonstrated that cryptographic hash functions specifically SHA-256 are essential tools for accurate file integrity verification that visual inspection cannot provide. Two files containing identical visible content produced completely different SHA-256 hashes, proving they contained hidden differences. Using the `sha256sum` command to generate hashes, the `>>` operator to save them, the `cat` command to display them, and the `cmp` command to compare them byte by byte, I confirmed the files were different at the first character of the first line. These skills are directly applicable to SOC analyst work including malware identification, digital forensics evidence validation, software authenticity verification, and continuous file integrity monitoring  all critical components of a robust organizational security posture.

---

## Skills Demonstrated

```
✅ SHA-256 hash generation using sha256sum
✅ File integrity verification methodology
✅ Hash comparison using cmp command
✅ Detection of hidden file modifications
✅ Linux command-line proficiency
✅ Understanding of cryptographic hash properties
✅ Real world application to malware detection
✅ Digital forensics chain of custody concepts
✅ NIST CSF alignment in security controls
✅ Professional security documentation
```

# File Permissions in Linux

## Project Description

In this project, I examined and modified file and directory permissions in a Linux environment to ensure that only authorized users could access sensitive resources. Using the `ls -la` and `chmod` commands, I identified permission misconfigurations across files and directories in the `/home/researcher2/projects` directory and applied the principle of least privilege to secure them. This project demonstrates my ability to manage Linux file system security — a core skill for any SOC Analyst or Security Engineer.

---

## Check File and Directory Details

To begin the investigation, I navigated to the `/home/researcher2/projects` directory and listed all files and directories, including hidden files, with their full permission details.

**Command used:**
```bash
cd /home/researcher2/projects
ls -la
```

**Output:**
```
total 32
drwxr-xr-x 3 researcher2 researcher2 4096 Jul 10 02:47 .
drwxr-xr-x 8 researcher2 researcher2 4096 Jul 10 02:47 ..
-rw-rw-r-- 1 researcher2 researcher2  46  Jul 10 02:47 project_k.txt
-rw-r----- 1 researcher2 researcher2  46  Jul 10 02:47 project_m.txt
-rw-rw-r-- 1 researcher2 researcher2  46  Jul 10 02:47 project_r.txt
-rw-r--r-- 1 researcher2 researcher2  46  Jul 10 02:47 project_t.txt
-rw--w---- 1 researcher2 researcher2  46  Jul 10 02:47 project_x.txt
-rw-rw-r-- 1 researcher2 researcher2  46  Jul 10 02:47 .project_x.txt
drwx--x--- 2 researcher2 researcher2 4096 Jul 10 02:47 drafts
```

The `-la` flags ensured that:
- `-l` displayed full permission details for every item
- `-a` revealed hidden files (files beginning with `.`) that a standard `ls` would not show

---

## Describe the Permissions String

Each item in the output begins with a 10-character permissions string. Here is how to read it using `project_k.txt` as an example:

```
-rw-rw-r--
│└─┘└─┘└─┘
│  │  │  └── Other:  r-- (read only)
│  │  └───── Group:  rw- (read and write)
│  └──────── User:   rw- (read and write)
└─────────── Type:   -   (regular file; d = directory)
```

**Permission characters explained:**

| Character | Meaning |
|-----------|---------|
| `r` | Read — can view file contents or list directory |
| `w` | Write — can modify file or create/delete in directory |
| `x` | Execute — can run file as program or enter directory |
| `-` | Permission not granted |

**File type character (first position):**

| Character | Meaning |
|-----------|---------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |

**Example breakdown — `drafts` directory:**
```
drwx--x---
│└─┘└─┘└─┘
│  │  │  └── Other: --- (no access)
│  │  └───── Group: --x (execute only — can enter directory)
│  └──────── User:  rwx (full access)
└─────────── Type:  d   (directory)
```

This revealed that the group had execute access to the `drafts` directory, which violated the security requirement that only `researcher2` should be able to access it.

---

## Change File Permissions

The organization requires that **other** should not have write access to any files. After reviewing the output, `project_k.txt` had write permission granted to other, which needed to be removed.

**Command used:**
```bash
chmod o-w project_k.txt
```

**Verification:**
```bash
ls -la project_k.txt
```

**Before:**
```
-rw-rw-rw- 1 researcher2 researcher2 46 Jul 10 02:47 project_k.txt
```

**After:**
```
-rw-rw-r-- 1 researcher2 researcher2 46 Jul 10 02:47 project_k.txt
```

**What changed:**
- Other's write (`w`) permission was removed
- User and group permissions remained unchanged
- Other now has read-only access — aligned with the principle of least privilege

---

## Change File Permissions on a Hidden File

The hidden file `.project_x.txt` had been archived and should no longer be writable by anyone. However, both the user and group still had write access. The requirement was:
- User: read only
- Group: read only
- Other: no access

**Command used:**
```bash
chmod u=r,g=r .project_x.txt
```

Alternatively using numeric notation:
```bash
chmod 440 .project_x.txt
```

**Verification:**
```bash
ls -la .project_x.txt
```

**Before:**
```
-rw-rw---- 1 researcher2 researcher2 46 Jul 10 02:47 .project_x.txt
```

**After:**
```
-r--r----- 1 researcher2 researcher2 46 Jul 10 02:47 .project_x.txt
```

**What changed:**
- User write (`w`) permission removed — now read only
- Group write (`w`) permission removed — now read only
- Other remains at no access
- File is now fully read-protected and cannot be modified by anyone

---

## Change Directory Permissions

The `drafts` directory and all of its contents should only be accessible to `researcher2`. The group had execute permission on the directory, which allowed group members to enter it and access its contents — a security misconfiguration.

**Command used:**
```bash
chmod g-x drafts
```

**Verification:**
```bash
ls -l
```

**Before:**
```
drwx--x--- 2 researcher2 researcher2 4096 Jul 10 02:47 drafts
```

**After:**
```
drwx------ 2 researcher2 researcher2 4096 Jul 10 02:47 drafts
```

**What changed:**
- Group execute (`x`) permission removed from `drafts`
- Group members can no longer enter the directory or access its contents
- Only `researcher2` (the owner) retains full `rwx` access
- This enforces the principle of least privilege at the directory level

**Why execute on a directory matters:**

| Permission on Directory | Effect |
|------------------------|--------|
| `r` | Can list contents with `ls` |
| `w` | Can create or delete files inside |
| `x` | Can enter with `cd` and access files inside |

Removing `x` from the group effectively locks the directory even if individual files inside had open permissions.

---

## Summary

In this project I audited and corrected file and directory permissions in a Linux environment using the `ls -la` and `chmod` commands. The following changes were made to align with the organization's security policy and the principle of least privilege:

| File / Directory | Issue Found | Action Taken | Result |
|-----------------|-------------|--------------|--------|
| `project_k.txt` | Other had write access | `chmod o-w project_k.txt` | Other restricted to read only |
| `.project_x.txt` | User and group had write access | `chmod u=r,g=r .project_x.txt` | Both restricted to read only |
| `drafts/` | Group had execute access | `chmod g-x drafts` | Group locked out completely |

**Key skills demonstrated:**
- Navigating the Linux file system using `cd` and `ls -la`
- Reading and interpreting 10-character permission strings
- Applying `chmod` using both symbolic (`u=r,g=r`) and numeric (`440`) notation
- Identifying permission misconfigurations that violate least privilege
- Securing hidden files and restricted directories
- Documenting security changes in a clear, audit-ready format

This type of permission auditing is a foundational task in Linux system hardening and is performed regularly by SOC Analysts, Security Engineers, and System Administrators to reduce the attack surface and prevent unauthorized access to sensitive resources.

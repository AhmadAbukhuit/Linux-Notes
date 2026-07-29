# Linux File Permissions and Access Control

In Linux, security and access control are built directly into the filesystem. Whether operating natively on Debian 13 or inside a WSL 2 environment, every file and directory is assigned specific access rights that dictate exactly who can read, modify, or execute it.

---

## Understanding the Permission String

When you list files using `ls -l` in your terminal, you will see a 10-character string (e.g., `-rwxr-xr-x` or `drw-r--r--`). 

The first character indicates the file type (`-` for normal file, `d` for directory). The remaining nine characters represent the permissions, broken down into three distinct sets of three characters:

1.  **User (u):** The owner of the file.
2.  **Group (g):** The user group assigned to the file.
3.  **Other (o):** Everyone else on the system (public access).

### The Core Permissions
The rights granted to each of the three categories are represented by three letters: **r**, **w**, and **x**. Their behavior changes slightly depending on whether they are applied to a file or a directory.

| Symbol | Name | Value | Effect on a File | Effect on a Directory |
| :--- | :--- | :--- | :--- | :--- |
| **`r`** | Read | 4 | Allows reading and copying of the file's contents. | Allows listing the directory's contents (e.g., using `ls`). |
| **`w`** | Write | 2 | Allows modifying, overwriting, or emptying the file (e.g., via Neovim). | Allows adding, removing, or renaming files within the directory. |
| **`x`** | Execute | 1 | Allows running the file as a program or script. | Allows entering the directory (e.g., using `cd`). |
| **`-`** | Denied | 0 | The specific permission is disabled. | The specific permission is disabled. |

---

## The `chmod` Command

The `chmod` (Change Mode) command is used to modify these access rights. There are two primary ways to use `chmod`: **Symbolic Mode** (using letters) and **Octal Mode** (using numbers).

**Basic Syntax:**
`chmod [options] [permissions] [file_or_directory]`

---

### Method 1: Symbolic Mode
Symbolic mode is highly flexible and is best used when you want to add or remove a specific permission without altering the existing rights. 

It follows a strict formula: `[Category] [Operation] [Permission]`

**1. Category (Who are you targeting?)**
| Symbol | Name | Description |
| :--- | :--- | :--- |
| **`u`** | User | The specific user who owns the file. |
| **`g`** | Group | The group that owns the file. |
| **`o`** | Other | Anyone who is not the owner or in the group. |
| **`a`** | All | Applies the change to `u`, `g`, and `o` simultaneously. |

**2. Operation (What action are you taking?)**
| Symbol | Action | Description |
| :--- | :--- | :--- |
| **`+`** | Add | Grants the permission. Existing permissions remain intact. |
| **`-`** | Remove | Revokes the permission. Existing permissions remain intact. |
| **`=`** | Set Exact | Overwrites current permissions, setting them exactly to what is specified. |

**3. Permission (Which right are you modifying?)**
Use `r` (read), `w` (write), or `x` (execute).

**Symbolic Mode Examples:**
*   `chmod u+x deploy_script.zsh`: Grants the **User** the right to **Execute** the file.
*   `chmod g-w config.txt`: Revokes **Write** access from the **Group**.
*   `chmod a+r public_doc.pdf`: Grants **Read** access to **All** users.
*   `chmod o-wx private_folder`: Revokes both **Write and Execute** access from **Others**.

---

### Method 2: Octal (Numeric) Mode
Octal mode is the absolute method. It uses a three-digit number to define the exact permissions for the User, Group, and Other simultaneously. It is faster than symbolic mode but completely overwrites the previous permissions.

Each digit is calculated by adding the numeric values of the desired permissions: **Read (4) + Write (2) + Execute (1)**.

**Permission Values Breakdown:**
| Octal Value | Binary | Symbolic | Granted Permissions |
| :--- | :--- | :--- | :--- |
| **0** | 000 | `---` | No permissions |
| **1** | 001 | `--x` | Execute only |
| **2** | 010 | `-w-` | Write only |
| **3** | 011 | `-wx` | Write and Execute (2 + 1) |
| **4** | 100 | `r--` | Read only |
| **5** | 101 | `r-x` | Read and Execute (4 + 1) |
| **6** | 110 | `rw-` | Read and Write (4 + 2) |
| **7** | 111 | `rwx` | Read, Write, and Execute (4 + 2 + 1) |

**Octal Mode Examples:**
*   `chmod 755 deploy_script.zsh`:
    *   User gets `7` (rwx): Read, Write, Execute.
    *   Group gets `5` (r-x): Read, Execute.
    *   Other gets `5` (r-x): Read, Execute.
    *   *Standard for executable scripts and directories.*
*   `chmod 644 config.txt`: 
    *   User gets `6` (rw-): Read, Write.
    *   Group gets `4` (r--): Read only.
    *   Other gets `4` (r--): Read only.
    *   *Standard for regular text files and documents.*
*   `chmod 700 private_keys`:
    *   User gets `7` (rwx): Full access.
    *   Group gets `0` (---): No access.
    *   Other gets `0` (---): No access.
    *   *Standard for highly sensitive files like Ed25519 SSH keys.*
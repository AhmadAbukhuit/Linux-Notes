# 📘 Section 01: Linux Overview & Foundations

Welcome to the **Overview & Foundations** module. This section establishes the core architectural principles of the Linux operating system, the structure of its filesystem, and its built-in security model.

Understanding these concepts is essential before executing administrative commands, managing services, or writing automation scripts.

---

## 📑 Contents in this Folder

| File | Title | Summary & Key Topics |
| :--- | :--- | :--- |
| **[`01_introduction.md`](./01_introduction.md)** | **Introduction to Linux** | Fundamental OS architecture, the role of the Linux kernel, system calls, the difference between shells and terminals, and Linux file type indicators (`-`, `d`, `l`, `c`, `b`, `s`, `p`). |
| **[`02_filesystem-hierarchy.md`](./02_filesystem-hierarchy.md)** | **Linux Filesystem Hierarchy (FHS)** | The unified directory tree starting from root (`/`), standard top-level directories (`/etc`, `/var`, `/home`, `/bin`, `/proc`, `/dev`), and crucial configuration files (`/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/hosts`). |
| **[`03_permissions.md`](./03_permissions.md)** | **Permissions & Ownership** | Decoding the 10-character permission string (`ls -l`), User/Group/Other triples (`u`, `g`, `o`), numeric/octal mode (`r=4, w=2, x=1`), modifying permissions with `chmod`, and changing ownership with `chown`. |

---

## 🧭 Learning Path

```text
+-----------------------------+
|    01_introduction.md       |  Understand OS layers, Kernel, Shell, and File Types
+--------------+--------------+
               |
               v
+-----------------------------+
| 02_filesystem-hierarchy.md  |  Navigate the Filesystem Hierarchy Standard (FHS)
+--------------+--------------+
               |
               v
+-----------------------------+
|     03_permissions.md       |  Master Linux ownership (chown) and permissions (chmod)
+-----------------------------+
```

---

## 🚀 Next Steps

Once you are comfortable with files, directory hierarchies, and access permissions, proceed to **[02-core-commands](../02-core-commands/README.md)** to learn everyday command-line workflows, service management, process triage, and Bash scripting.

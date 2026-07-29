# Linux Commands Reference

Most commands follow a simple pattern of syntax :

```bash
command [options] [arguments]
```

### Options:
*   Used to change the behavior of commands
*   More than one option can be put together for one command

### Arguments:
*   Used to specify something for commands to work with

## Navigation and Basic Utilities
*   `pwd`: Use to print your current path in directories.
*   `cd [options] [arguments]`: Use to navigate system files.
    *   `Cd ..`: Return to the previous file.
    *   `Cd`: Back to the main file.
*   `ls [options] [arguments]`: Use to list the contents of a file.
    *   Available options include `l`, `a`, and `h`.
*   `man [command]`: Use to display details of the requested command.
*   `clear`: Use to Delete commands from the terminal screen.

## File and Directory Management
*   `mkdir [name]`: Create new directory.
*   `rmdir [name]`: remove empty directory.
*   `touch [name]`: Create new file.
*   `rm [options] [name]`: remove file or directory.
    *   Available options include `f` and `r`.
*   `echo [string]`: Use to print on terminal.
*   `cp [options] [source] [destination]`: Use to copy files or directories from source to destination.
*   `mv [options] [source] [destination]`: Use to move files or directories from source to destination, or swap a files or directories name.
*   `file [options] [name]`: Use to display type and information about the files and directories.
*   `locate [options] [name]`: Use to file search.
*   `find [options] [name]`: Use to file search.

## Viewing File Contents
*   `cat [options] [name]`: View the contents of the file.
*   `more [options] [name]`: View the contents of the file in parts.
*   `less [options] [name]`: View the contents of the file in parts, less is better than more because there are more options.
*   `head [options] [name]`: Show the first part of the file.
    *   `-n num`: Shows the first num lines of the file.
*   `tail [options] [name]`: Show the last part of the file.
    *   `-n num`: Show the last num lines of the file.

## Archiving and Compression
*   `tar -cvf [name] [filename]`: Archive files with ".tar" extension.
*   `tar -xf [name]`: Unarchive files with ".tar" extension.
*   `zip [name] [filename]`: Archive files with ".zip" extension.
*   `unzip [name] [filename]`: Unarchive files with ".zip" extension.
*   `7z [command] [name] [filename]`: Archive files in a high compression ratio format.
    *   Commands include `a`, `d`, `e`, `l`, and `x`.

## System and Network
*   `date`: Display system date and time.
*   `history`: Display order history on the system.
*   `shutdown [options] [time] "message"`: It is used to shut down the system in a safe way and notifies users of this.
*   `ifconfig`: View network information.
*   `ssh username@IP`: Connect to another device using a secure channel.

## Privilege and Account Management
*   `sudo [option] [command]`: Execute command in higher privilege.
*   `su [option] [username]`: Change to anther user in system.
*   `passwd`: Change my user password.
*   `exit`: Log out from my user account.
*   `useradd [options] [username]`: Create new user account.
*   `userdel [options] [username]`: Delete user account.
*   `groupadd [options] [groupname]`: Create new group of users.
*   `groupdel [options] [groupname]`: Delete group.

## Text Processing and Filtering
*   `grep [option] [string] [file]`: search for matching patterns in a file.
*   `awk [option] '/string/ {action}' [file]`: performs the pattern/action statements once for each record in a file.
*   `Sed [option] '[action] /st1/st2 /[] [file]`: mostly used to replace the text in a file.
*   `tr "char1 ""char2 " [file]`: Use to translates or deletes characters.
*   `cut [option] [file]`: Use to cut out sections of a specified file.
*   `wc [option] [file]`: Used to find out number of lines, word count, byte and characters count in the files.
*   `uniq [option] [file]`: Used to filter out the repeated lines in a file.
*   `cmp [option] [file1] [file2]`: Used to compare the two files byte by byte and helps you to find out whether the two files are identical or not.
*   `Paste [option] [file1] [file2]`: Used to join files horizontally (parallel merging).
*   `hd [option] [file]`: Used to filter and display the specified files, or standard input in a human readable specified format.
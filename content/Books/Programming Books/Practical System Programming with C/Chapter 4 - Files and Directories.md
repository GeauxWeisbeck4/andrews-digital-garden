---
id: 01JBMM3007D3K7D8AXAPET35GB
modified: 2024-11-01T15:44:48-04:00
title: Chapter 4 - Files and Directories
tags:
  - linux
  - unix
  - systems-programming
  - programming
  - books
---
# 4. Files and Directories

Sri Manikanta Palakollu[1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Aff2) 

(1)

freelance, Hanuman Junction, Hanuman Junction, 521105, Andhra Pradesh, India

You have learned about command-line topics in Linux. You were introduced to multithreading and the practical implementation of multithreading in C. You saw API support in C to develop efficient applications. This chapter discusses files and directories. By the end of this chapter, you should have a deep knowledge of the core concepts regarding Linux files, directories, and storage mechanisms. The chapter’s code examples and discussions cover the following topics.

- File system
    
- Inodes and file metadata
    
- Inode storage mechanisms
    
- System calls and I/O operations for files
    
- Systems calls for file permissions
    
- File permission checks
    
- Soft links
    
- Hard links
    
- System calls for directories
    

## File Systems

Storage is one of the most essential components of an OS. The storage system must have a well-organized structure, and it must be easy to access the content. In Linux/Unix-based systems, everything is a file, which maintains the consistency that makes the files easy to access. The file system in Unix is groups files into folders based on purpose and use in a well-organized way. For most beginners, this seems intimidating. But once you understand the purpose of each directory that is part of the Unix file system, then it becomes easy to use and work with it.

The Unix file system looks like a tree structure. It consists of several directories, and each directory consists of several specific files and subdirectories. In Unix, all the directories are in the root directory (/). The entire Unix file system structure is represented in Figure [4-1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig1).

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig1_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig1_HTML.jpg)

Figure 4-1

Linux file structure

- **/ (the root directory)**
    
    - This directory is called the _root directory_.
        
    - All the files and directories in your Linux or Unix system are grouped into this root directory.
        
    - In this directory, only root users have permission to write.
        
    
- **/bin (binaries)**
    
    - This directory contains all the essential binaries in the operating system.
        
    - The most frequently used Linux commands in a single-user mode are located in this directory.
        
    - Examples of available commands are cd, mkdir, ls, mv, and cp.
        
    - Shells like bash, ksh, csh, zsh are located in this directory only. (i.e., (/bin/bash, /bin/sh, etc.).
        
    
- **/sbin (system binaries)**
    
    - This directory is similar to the /bin directory, but it contains the system administration binaries, which are executed by the root user.
        
    - The commands and programs available in this directory are only executed by the superuser.
        
    - The most common programs available in /sbin are ifconfig, iptables, and reboot.
        
    
- **/boot (boot files)**
    
    - This directory contains all boot loader–related files, which are very important for booting an operating system.
        
    - All the files in this directory are static boot files. This directory does not contain any boot configuration files.
        
    - An example of a boot loader file is a GRUB loader file, which boots an operating system when you power-on your laptop or desktop.
        
    
- **/opt (optional packages)**
    
    - This directory consists of all the files that are not part of the default installation.
        
    - Third-party software installed on your system that did not come as a default installation on Unix/Linux. Proprietary software installation files are placed in this directory.
        
    - Examples of software installed on Unix/Linux from third-party sources include Apache server and Apache Tomcat server.
        
    
- **/dev (device files)**
    
    - The files in this directory represent the hardware device files.
        
    - As with everything in Linux, devices are represented by a directory or a file; so, all device-supported files for the system are placed here.
        
    - This directory consists of special device files that come with the installation of the operating system. They help the operating system to support all types of devices detected while running the operating system.
        
    
- **/home (home directory)**
    
    - This is the home directory, which contains each user’s files.
        
    - This directory consists of the user’s personal data and configuration files. The configuration files in this directory vary from user to user.
        
    - If your system user name is Alex, then you have a home folder located at /home/Alex. The number of user accounts on your system equals the number of subdirectories present.
        
    
- **/media (removable media)**
    
    - This directory consists of all the removable device directories.
        
    - When an external removable device is mounted in a Linux system, then automatically, a new directory is created under this directory.
        
    - When a USB is inserted to a laptop/PC that is running on a Linux/Unix-based operating system, then the /media directory creates a directory for the removable USB. This directory contains all the removable media files that are automatically created by the operating system.
        
    
- **/mnt (mount directory)**
    
    - This directory consists of all the mounted files in a system.
        
    - Suppose that your PC is dual booted (_dual boot_ means you can have two operating systems on one PC). The number is not restricted. You can have any number, but hard disk space is limited. In a dual boot PC, all the other operating system mounted files are placed in this directory.
        
    - System administrators use this directory to unmount a mounted file system. Normal users can’t mount a filesystem without root privileges.
        
    
- **/etc (configuration files)**
    
    - This directory consists of all the configuration files that are used by all the programs in a Linux operating system.
        
    - System-level configuration files are placed in this directory. The user-level configuration files are placed in the user-level home directory.
        
    - Startup and shutdown scripts are located in this directory.
        
    - You can configure it with editors for your own use; for instance, the configuration of a LAMP server.
        
    
- **/lib (system libraries)**
    
    - This directory contains the essential libraries needed by the binaries that are in the /bin and /sbin directories.
        
    
- **/usr (user programs and data)**
    
    - This directory contains all the binary files and applications that are used by the user.
        
    - /usr/lib contains the binaries for /bin and /sbin.
        
    - Applications installed from the source are placed in the /usr/local directory.
        
    - /usr/sbin contains the binary files for system administrators. If you are unable to find the required files in this directory, go to the root level /sbin directory.
        
    - This directory contains the source code for second-level programs, which do not come from the default installation.
        
    - All the files in this folder have read-only access because they are system-related binary files.
        
    
- **/root (root home directory)**
    
    - This is the home directory for the root user; it is not a system root directory.
        
    - Most people confuse / and /root. The major difference is that / is a system-level root directory. And /root is the user-level root directory.
        
    
- **/var (variable data files)**
    
    - This directory consists of all the user data files in a system. This data refers to the system data.
        
    - This directory contains log files under /var/log; packages and database files under /var/lib; and temporary files under /var/tmp.
        
    
- **/srv (service data files)**
    
    All the internal operating system service data files are located in this directory; for instance, files that are related to local servers are found under this directory.
    
- **/tmp (temporary files)**
    
    - All the temporary files created by the system or user are placed in this directory. If the user has root privileges, then he/she can put the temporary files in any location, but /tmp is the recommended and system-assigned one.
        
    - All the system-level temporary files are stored in this directory.
        
    - The files in this directory are deleted when the system is rebooted. The deletion of temporary files is dependent on the Linux distribution, however, because some Linux distros do not delete the temporary files in the system after every reboot.
        
    - Users can manually delete the files in the tmp directory.
        
    

It seems a bit hard to remember everything, but once you start using these directories, it becomes easier. Each directory stores specific programs and utility applications. Unlike Windows, Unix-based systems don’t have drive letters, like C: drive and D: drive. You can create partitions in Linux, but all the partitioned disk space is packed into a tree-like structure that has a root (/) directory.

### File Metadata and Inodes

Everything is a file descriptor in Linux. A _file descriptor_ is a number that uniquely identifies a file that is open on an operating system. The file descriptor contains information about the opened file. Every file has certain key attributes to identify its properties. Regular files include images, audio, video, and other raw files that usually have metadata associated with them. All the file attributes are stores in an inode.

An _inode_ is a data structure that tracks all the information about a file. An inode plays a very crucial role in Unix-based operating systems. When a user wants to access a file, the operating system first searches for the inode number in an associated inode table. The storage of inode numbers in an operating system and finding a particular file using the path is explained in Figure [4-3](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig3). Every inode has a unique number. An inode number is the index number that is associated with each inode. OS searches for an inode number come from an inode table. The Unix storage system is a bit different from the Windows storage system, but it is more efficient. The structure of an inode is represented in Figure [4-2](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig2).

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig2_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig2_HTML.jpg)

Figure 4-2

Inode structure

- **Size** stores the file size.
    
- **Mode** stores information on the file’s permissions and the type (i.e., directory, file, device directory, etc.).
    
- **Owner information** points to the person who created the file.
    
- **Permissions** contain all the permissions levels for a file (i.e., user, group, other users).
    
- **Location** describes the exact location of the file on the operating system.
    
- **UID** is short for _user ID_. It stores the user ID of the currently working user and represents the owner of the file.
    
- **GID** stands for _group ID_. It stores the group ID of the file that belongs to and represents the group owner.
    
- **Timestamp** stores the inode creation time and when the file was modified.
    
- **Access control** contains information on the special privileges given to groups and other users (the outside real-world users).
    
- **Direct block**
    
    - Linux usually follows the file system ext2, et3, et4; but for now, we discuss the ext2 file system, which is popular. In the ext2 file system, an inode consists of 12 direct block pointers.
        
    - The first 12 block pointers are direct.
        
    - The direct block directly points to the file data, as shown in Figure [4-2](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig2).
        
    - In the direct block system, 12 blocks are reserved for storing the file pointer’s address and directly pointing to the data/file.
        
    - Each direct block points to a file that is 4 KB. In total, direct blocks can store 48 KB.
        
    - Direct block storage is very limited (i.e., 48 KB). It can’t point to large data files or directories. Indirect blocks overcome this issue.
        
    
- **Indirect block**
    
    - It points to the files or directories that are greater than 4 KB and less than or equal to 4 MB.
        
    - It is more advanced than the direct block method. It creates 1024 different blocks internally. Each block stores 4 KB of data, which is very small. Data pointers point to the 1024 blocks. Since each block stores 4 KB of data, this results in a total 4 MB of data. This is called the _indirect block mechanism_.
        
    - The Unix/Linux system is intelligent at detecting and effectively monitoring data. If the size of a file/directory is more than 4 MB, it automatically moves to the double indirect block method.
        
    - Data pointers internally point to the 1024 block pointers that store the file data.
        
    
- **Double indirect block**
    
    - It creates 1024 different blocks to point the data. Each block can store 4 MB of data. Internally, it points to the indirect block address, which can point up to 4 MB.
        
    - It can point up to 4 GB of data. If the file or directory is greater than 4 GB, it automatically transfers the pointer data to the triple indirect block.
        
    - It internally points to 1024 indirect block pointers.
        
    
- **Triple indirect block**
    
    - It creates 1024 different blocks to point the data. Each block can store 4 GB of data. Internally, it points to the double indirect block address, which can point up to 4 GB.
        
    - It can point to as much as 4 TB of data. It internally points to the 1024 double indirect block pointers.
        
    

Figure [4-3](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig3) shows an inode pointing mechanism structure that represents accessing a file named script.ts on the operating system. Suppose that a user wants to access a file on the operating system at /root/Desktop/WebDev/script.ts.

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig3_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig3_HTML.jpg)

Figure 4-3

Structure of inode for file storage mechanism in Linux

Initially, the root directory has a specific inode number that points to the root number. It consists of several files and folders. In our situation, the root directory has home, desktop, documents, and downloads directories. The desktop directory has an inode number that consists of several lists of files and folders (i.e., Python, Web Dev, Courses directories, and a helloworld.c program file).

Each directory and program file has its own inode number. When I navigate to the Web Dev directory, it has a separate inode number that points to another set of files and directories (that include index.html, style.css, script.js, script.ts), which have different inode numbers. The searched file is available in the Web Dev directory. If the given file name is not available in the specified location, an error is immediately thrown. All the inode numbers are internally connected to provide better and faster access to files and directories.

When you enter a directory and input the ls command in the command line, all the files and directories in that directory are displayed based on the inode number. This happens because of all the inode numbers of files and directories in that directory point to an array of inode numbers that point to the parent directory. This is represented in Figure [4-3](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig3); for example, the desktop inode is pointing to six different inode numbers. Those are the files and directories in the desktop directory.

## System Calls and I/O Operations for Files

The most common operations that you can perform on files are read, write, delete, and modify the content. This section explains dealing with file manipulation programmatically. Our program uses core system calls to manipulate the tasks. To get more information on a system call, you can use the man command. For example, man creat gives complete information on the creat system call (see Figure [4-4](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig4)).

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig4_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig4_HTML.jpg)

Figure 4-4

Man command information on creat system call

These system calls set an error number if they fail to perform the operation. The error numbers analyze why the system call is unable to perform a particular activity and may quickly debug the application.

To get the error code for your application, you need to use the error function explicitly to print the message to the console. Table [4-1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Tab1) describes some of the most useful error codes.

Table 4-1

Useful Error Codes

|Error Number|Error Code|Description|
|---|---|---|
|1|EPERM|Operation Not Permitted|
|2|ENOENT|No Such File Or Directory|
|3|ESRCH|No Such Process|
|4|EINTR|System Call Interrupted|
|5|EIO|I/O Error|
|6|ENXIO|No such device or address|
|8|ENOEXEC|Exec Format Error|
|9|EBADF|Bad File Number|
|10|ECHILD|No child processes available|
|11|EAGAIN|Try again|
|12|ENOMEM|Out of Memory|
|13|EACCES|Permission Denied|
|14|EFAULT|Bad Address|
|16|EBUSY|Device or Resource Busy|
|17|EEXIST|File Exist|
|20|ENOTDIR|Not a Directory|
|21|EISDIR|Is a Directory|
|22|EINVAL|Invalid Argument|
|23|ENFILE|File Table Overflow|
|24|EMFILE|Too many open files|
|27|EFBIG|File too large|
|28|ENOSPACE|No Space Available on device|
|29|ESPIPE|Illegal Seek|
|30|EROFS|Read-Only File System|
|32|EPIPE|Broken Pipe|
|33|EDOM|Math argument out of the domain|
|34|ERANGE|Math results not representable|
|39|ENOTEMPTY|Directory Not Empty|
|40|ELOOP|Too many symbolic links occurred|
|62|ETIME|Timer Expired|
|64|ENONET|The machine is not available in the network|
|65|ENOPKG|Package is not available|
|71|EPROTO|Protocol Error|
|86|ESTRPIPE|Stream Pipe Error|
|87|EUSERS|Too many users|
|91|EPROTOTYPE|Protocol error for socket|

The following are the system calls that are available for file operations.

- creat
    
- open
    
- close
    
- read
    
- write
    

### creat

This system call creates a new empty file with a system call. It is available in the fcntl.h library, which is a file handling library for Unix and Linux. The return type for this function is an integer. If file creation is successful, it returns a non-negative integer. If the creation of the file fails, it returns –1.

The following shows the syntax.

**int creat(char *filename, mode_t mode);**

- The first parameter in the creat function is the name of a file.
    
- The second parameter, mode, deals with the permissions of the file. The permission modes are different from normal Linux file system permissions. There are various modes available for this flag, but the following are the most common modes.
    
    - O_RDONLY: If you set this flag mode to the creat function, the file has read-only permission.
        
    - O_WRONLY: This mode gives write permissions.
        
    - O_RDWR: This mode gives both read and write permissions.
        
    - O_EXCL: This flag mode prevents the creation of a file if it already exists.
        
    - O_APPEND: This mode appends the content to existing file data without overriding it.
        
    - O_CREAT: This flag mode creates a file if it does not exist.
        
    

If you want to use multiple modes at the same time, you can use the bitwise OR operator.

Here’s an example.

**#include<stdio.h>**

**#include<fcntl.h>**

**int main(){**

   **int file_descriptor;**

   **char filename[255];**

   **printf("Enter the filename: ");**

   **scanf("%s", filename);**

   **// Setting Permission to Read and Write Access for the file.**

   **file_descriptor = creat(filename, O_****RDWR** **| O_CREAT);**

   **if(file_descriptor != -1){**

       **printf("File Created Successfully!");**

   **}else{**

       **printf("Unable to Create the File.");**

   **}**

   **return 0;**

**}**

A file descriptor is an integer value that identifies the open file in a process. This program creates a new file with the given permissions set.

### open

The open system call function opens a file and can perform read and write operations based on the mode set to the function. An open system call can also create a file. If the specified filename is not available, then it automatically creates a new file with the given name. The return type of this function is an integer. If the file opens successfully, it returns a positive integer value; otherwise, it returns –1.

The following shows the syntax.

**int open(const char *filepath, int flags, ...);**

- The first parameter deals with the absolute path of a file that you want to open.
    
- The flags that are passed as a second argument are O_RDONLY, O_WRONLY, O_RDWR, and so forth.
    

Here’s an example.

**#include<stdio.h>**

**#include<fcntl.h>**

**int main(){**

   **int file_descriptor;**

   **char filename[255];**

   **printf("Enter the filename: ");**

   **scanf("%s", filename);**

   **// Setting Permission to Read Only for the file.**

   **file_descriptor = open(filename, O_RDONLY);**

   **/***

   **On Success: It returns any value other than -1.**

  ***/**

   **if(file_descriptor != -1){**

       **printf("%s Opened Successfully!",filename);**

   **}else{**

       **printf("Unable to Open %s",filename);**

   **}**

   **return 0;**

**}**

This program prints a statement on whether the given file is open or not.

### close

This system call closes the file descriptor that was created to open, create, or read the contents in a file. The return type of this function is an integer. If the file descriptor is closed successfully, it returns 0; otherwise, it returns –1.

The following shows the syntax.

**int close(int file_descriptor);**

file_descriptor is an integer value that identifies the open file in a process.

Here’s an example.

**#include<stdio.h>**

**#include<fcntl.h>**

**int close(int file_descriptor);**

**int main(){**

   **int file_descriptor;**

   **char filename[255];**

   **printf("Enter the filename: ");**

   **scanf("%s", filename);**

   **// Setting Permission to Read Write for the file.**

   **file_descriptor = open(filename, O_****RDWR****, 0);**

   **if(file_descriptor != -1){**

       **printf("File Opened Successfully!\n");**

   **}else{**

       **printf("Unable to Open the File.\n");**

   **}**

   **int close_status = close(file_descriptor);**

   **// Checks the condition and prints the appropriate statement.**

   **/***

    **Success: 0**

    **Error: -1**

    ***/**

   **if(close_status == 0){**

       **printf("File Descriptor is closed Successfully\n");**

   **}else{**

       **printf("File Descriptor is not closed\n");**

   **}**

   **return 0;**

**}**

This program opens a file and closes the file descriptor after the task is done. After the file descriptor is opened and work is done, it is a good programming practice to close the descriptor.

### read

This function system call reads the content of a file that was indicated by a file descriptor. The return type of this function is an integer. It returns –1 if an error occurs or when any signal interrupt occurs during a read operation. A successful read of a file returns the number of bytes read during the operation.

The following shows the syntax.

**size_t read (int file_descriptor,**

             **void* buffer,**

             **size_t size);**

- file_descriptor is a unique integer value that identifies the open file in a process.
    
- The **buffer** argument reads the file data.
    
- **size** is the third argument indicates the size of the buffer that you want to read from the file.
    

Here’s an example.

**#include<stdio.h>**

**#include<stdlib.h>**

**#include<unistd.h>**

**#include <fcntl.h>**

**int main() {**

  **int file_descriptor, size;**

 **char filename[255];**

 **char *content = (char *) calloc(100, sizeof(char));**

 **printf("Enter the filename to read:");**

 **scanf("%[^\n]%*c",filename);**

 **file_descriptor = open(filename, O_RDONLY);**

 **// Program exit if the given file is not found.**

 **if (file_descriptor == -1) {**

      **perror("File Not found.");**

      **exit(1);**

   **}**

 **// read the Content from a given file descriptor.**

 **size = read(file_descriptor, content, 100);**

 **printf("Number of bytes returned are: %d\n", size);**

 **content[99] = '\0';**

 **printf("File Content: %s\n", content);**

 **// Closes the file descriptor.**

 **close(file_descriptor);**

 **return 0;**

**}**

This program prints the number of bytes that were read and then prints the content.

### write

This function writes content to a given file descriptor. The return type of this file is an integer. It returns –1 for an error or if any signal interrupt is raised; otherwise, it returns the number of bytes that are returned to a file.

The following shows the syntax.

**size_t write (int file_descriptor,**

              **void* buffer,**

              **size_t size);**

This function syntax is the same as the read function. But the key difference is that it writes the content to a file using the buffer. The read function reads the content from a file using a buffer.

Here’s an example.

**#include<stdio.h>**

**#include<unistd.h>**

**#include<stdlib.h>**

**#include<string.h>**

**#include <fcntl.h>**

**int main() {**

  **char filename[255];**

 **// Asking the Input from a User.**

 **printf("Enter the filename to open:\n");**

 **scanf("%[^\n]%*c", filename);**

 **int file_descriptor = open(filename, O_WRONLY | O_CREAT, 0777);**

 **if (file_descriptor == -1) {**

    **perror("File not Found.!");**

    **exit(1);**

 **}**

 **char content[1024];**

 **printf("Enter the content to write on a given file: ");**

 **// User Input to write into a File.**

 **scanf("%[^\n]%*c", content);**

 **int size = write(file_descriptor, content, strlen(content));**

 **printf("%d", size);**

 **close(file_descriptor);**

 **return 0;**

**}**

This code opens a given file and asks the user to enter the content that they want to write to it. The write function writes content and prints the number of bytes written to a file.

### Append Operations in Files Using System Calls

A write system call writes the content to the given file descriptor. But the O_RDWR flag mode overrides the content in an existing file. If you want to add more content to a file without overriding the existing content, the O_APPEND flag adds the new content at the end of the file without overriding any of the existing content.

Here’s an example.

**#include<stdio.h>**

**#include<unistd.h>**

**#include<stdlib.h>**

**#include<string.h>**

**#include <fcntl.h>**

**int main() {**

  **char filename[255];**

 **printf("Enter the filename to open:");**

 **scanf("%[^\n]%*c", filename);**

 **int file_descriptor = open(filename, O_WRONLY | O_CREAT | O_APPEND, 0777);**

 **if (file_descriptor == -1) {**

    **perror("File not Found.!");**

    **exit(1);**

 **}**

 **char content[1024];**

 **printf("Enter the content to write on a given file: ");**

 **scanf("%[^\n]%*c", content);**

 **int size = write(file_descriptor, content, strlen(content));**

  **printf("%d %lu %d\n", file_descriptor, strlen(content), size);**

  **close(file_descriptor);**

 **return 0;**

**}**

This program appends content to an existing file without overwriting it.

## File Permissions

In Chapter [1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_1_Chapter.xhtml), you saw how to change the permissions of a file using the Linux commands. This section deals with changing file permissions programmatically and identifying all the permissions given to a particular file by using built-in attributes. These attributes are properties that check the permissions using R_OK, W_OK, F_OK, and X_OK.

### chmod Function to Change Permissions

The chmod system call is available in the fcntl.h library. It changes the file permissions using the C program. The return type of this function is an integer. It returns 0 if it is successful and –1 if failure occurs.

The following shows the syntax.

**int chmod(char *filepath, int mode);**

- filepath is the first argument usually takes the complete file path with the respective file name as an argument.
    
- mode takes the new permission values as an argument. The value passed changes the file’s permissions.
    

Here’s an example.

**#include<stdio.h>**

**#include<fcntl.h>**

**#include<stdlib.h>**

**int chmod(char *path, mode_t mode);**

**int main(){**

   **int permission_status;**

   **mode_ t new_permission_value;**

   **char filepath[100];**

   **// Taking the Input from the user**

   **printf("Enter the filename with path: ");**

   **scanf("%[^\n]%*c", filepath);**

   **printf("Enter the new permission set: ");**

   **// Permission Set value starts with 0.**

   **// Eg: if i want to set 444 to a particular file then i need to give like 0444.**

   **scanf("%d", &new_permission_value);**

   **// Setting the Permissions**

   **permission_status = chmod(filepath, new_permission_value);**

   **// 0 ---> On Success || -1 ---> On Failure.**

   **if (permission_status == 0){**

        **printf("New permissions are Setted Successfully.!");**

   **}else{**

       **printf("Permissions Changed Successfully");**

   **}**

   **return 0;**

**}**

This program changes the permissions of a file. It returns a success statement if successful and returns a failure message if it is unable to change the permissions.

### File Permissions Check

You can check a file’s permissions with the access function in C, which is available in the unistd.h library. This return type of this function is an integer. It returns 0 if successful and –1 if failed.

The following shows the syntax.

**int access(const char *filepath, int amode);**

- filepath is the first argument that takes the complete file path (i.e., absolute path.
    
- The **amode** flag checks the permissions of a given file. The flags that pass for the second argument are any of the following.
    
    - R_OK: This flag tests the read permissions of a file.
        
    - W_OK: This flag tests the write permissions of a file.
        
    - F_OK: This flag tests whether a file exists or not.
        
    - X_OK: This flag tests the execute permissions of a file.
        
    

Now let’s look at a simple example program that determines the read and write access permissions of a file.

**#include<stdio.h>**

**#include<unistd.h>**

**int main(){**

   **char filepath[100];**

   **// Taking the Input from the user**

   **printf("Enter the filename with path: ");**

   **scanf("%[^\n]%*c", filepath);**

   **int read_status, file_status, write_status;**

   **file_status = access(filepath, F_OK);**

**/***

   **Returns**

   **-1 ----> If File Doesn't Exist**

    **0 ----> If File Exists**

***/**

   **if(file_status == -1){**

      **printf("%s File does not exist in Location.\n", filepath);**

      **_exit(0);**

   **}**

   **read_status = access(filepath, R_OK)** **;**

   **write_status = access(filepath, W_OK);**

   **// Checks for the both Read and Write Access**

   **if(read_status == 0 && write_status == 0){**

       **printf("%s File has both read and write permissions\n", filepath);**

   **}else if(read_status == 0 && write_status == -1){**

       **// If file has only read access then**

       **printf("%s File has only read permissions\n", filepath);**

   **}else if(read_status == -1 && write_status == 0){**

       **// If file has only write access then**

       **printf("%s File has only write permissions\n", filepath);**

   **}else{**

       **// If file does not have both read and write access then**

  **printf("%s File has no read and write permissions", filepath);**

   **}**

   **return 0;**

**}**

This program tests the read and write access of a file and prints the output (i.e., read or write). You can modify the code to test execute permissions as well.

## Soft and Hard Links

A link is a pointer to a file in the Unix system. In Linux, everything is considered a file, which has an inode number. A link acts as a shortcut to quickly access it. This happens because the link either points to the original file or its inode number. This helps the link provide faster access to the content. There are two types of links available in Linux: soft links and hard links.

### Soft Links

A _soft link_ links to a file, which contains data. There may be more than one soft link for a single file.

- It is also called a _symbolic link_ .
    
- Different inode numbers and permissions are set for a link. A soft link offers easy access to the original file because it directly points to the original.
    
- Soft links can be created for files and directories in the system.
    
- Permissions are not updated in a soft link. Permissions are updated in the main file, but the soft link permissions are not updated. This is one of the weird behaviors of soft links.
    
- The changes that are made to the original file are updated. All the changes made to the soft link file are updated to the main file.
    

A diagram of symbolic links is shown in Figure [4-5](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig5).

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig5_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig5_HTML.jpg)

Figure 4-5

Soft link internal connection

#### Creating a Soft Link Using the Command Line

The creation of a soft link is done with the ln command and some extra flags. Let’s look at creating a soft link.

ln -s <file_name> <softlink_name>

Note

The soft link name is your choice; there are no restrictions on for user’s name.

Figure [4-6](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig6) shows an example.

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig6_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig6_HTML.jpg)

Figure 4-6

Soft link creation using command line

#### Unlinking a Soft Link

When a soft link is created, a link file is too. There are two ways to remove these links from the file.

- Delete the link file
    

rm <soft_link_name>

- Unlink the file
    

unlink <soft_link_name>

After the unlinking action is done, the soft link is no longer available in the system. Let’s closely look at the example shown in Figure [4-7](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig7).

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig7_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig7_HTML.jpg)

Figure 4-7

Unlinking the soft link using a command line

#### Creating a Soft Link Using System Calls

You can programmatically create symbolic or soft links. In the unistd.h library, there is a system call named symlink that creates symbolic links effectively. The symlink function returns an integer value. It returns 0 on the successful creation of a symbolic link; it returns –1 if any failure occurs.

The following shows the syntax.

**int symlink(const char *filepath,**

            **const char *linkname);**

- **filepath** takes its name as an argument where the file is located in the system.
    
- **linkname** takes the link name as an argument. You can also use the path where you want to store it.
    

Now let’s programmatically create a symbolic link in a simple way.

**#include<stdio.h>**

**#include<unistd.h>**

**int main(){**

   **int link_status;**

   **char filepath[50], linkname[50];**

   **// Taking User Input for file path**

   **printf("Enter the filepath: ");**

   **scanf("%[^\n]%*c", filepath);**

   **printf("Enter the linkname: ");**

   **scanf("%[^\n]%*c", linkname);**

   **link_status = symlink(filepath,linkname);**

   **// 0 ---> On Success || -1 ---> If Any Error Occurs**

   **if(link_status == 0){**

       **printf("Soft link is Created Successfully.!");**

   **}else{**

       **printf("Unable to Create the Link.");**

   **}**

   **return 0;**

**}**

This program creates a soft link and returns a success statement if there is success and failure messages if any error occurs.

#### Unlinking Using a System Call

The unlink function unlinks a link. It is available in the unistd.h library. It returns an integer value: 0 if successful and –1 if any failure occurs.

The following shows the syntax.

**int unlink(const char *pathname);**

- pathname takes the link path as an argument and unlinks the pointer from the original file.
    

Let’s look at unlinking a pointer from a file in a simple way.

**#include<stdio.h>**

**#include<unistd.h>**

**int main(){**

   **int unlink_status;**

   **char linkname[100];**

   **// Taking the Link name as Input from the user to unlink**

   **printf("Enter the link name to unlink:");**

   **scanf("%[^\n]%*c", linkname);**

   **unlink_status = unlink(linkname);**

   **// 0 --->On Success || -1 ---> Failure.**

   **if(unlink_status == 0){**

       **printf("File is unlinked Successfully.!");**

   **}else{**

       **printf("Unable to unlink the file.");**

   **}**

   **return 0;**

**}**

This program unlinks the file from the pointer. It does the same as the command rm, and unlink does. On successful unlinking of a pointer, it returns the successful message. On an unsuccessful unlink, it returns a failure message.

### Hard Links

A _hard link_ is a mirror copy of the original file. If you accidentally delete the original file, the data remains in the hard link file, since it is a mirror copy of the original file. The following are some of the implications.

- The data updated on the original file is reflected on the hard link.
    
- It works only on a single file system, which means that it can’t create hard links for other operating system file systems.
    
- It can’t link to directories.
    
- It has the same inode number and permissions as the original file.
    

Figure [4-8](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_4_Chapter.xhtml#Fig8) is a diagram of a hard link.

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig8_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig8_HTML.jpg)

Figure 4-8

Hard link internal connection

#### Creating a Hard Link Using the Command Line

The creation of a hard link is done with the ln command. The link and the original file have the same data, permissions, and inode number.

ln <filename> <hard_link_name>

Here’s an example.

#### Creating a Hard Link Using a System Call

The creation of a hard link is done with a link system call. This system call function is available in the unistd.h library. It returns an integer value: 0 for a success and –1 for a failure.

![../images/497677_1_En_4_Chapter/497677_1_En_4_Fig9_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_4_Chapter/497677_1_En_4_Fig9_HTML.jpg)

Figure 4-9

Hard link creation using command line

The following shows the syntax.

**int link(const char *filepath,**

         **const char *linkname);**

- **filepath** takes its name as an argument where the file is located in the system.
    
- **linkname** takes the link name as an argument. You can also provide the path where you want to store it.
    

Here’s an example.

**#include<stdio.h>**

**#include<unistd.h>**

**int main(){**

   **int link_status;**

   **char filepath[50], linkname[50];**

   **printf("Enter the filename: ");**

   **scanf("%[^\n]%*c", filepath);**

   **printf("Enter the linkname: ");**

   **scanf("%[^\n]%*c", linkname);**

   **link_status = link(filepath, linkname);**

   **// Hardlink be Created.**

   **// 0 ---> Successful || -1 ---> Failure.**

   **if(link_status == 0){**

       **printf("HardLink is Created Successfully.!");**

   **}else{**

       **printf("Unable to Create the Hard Link.");**

   **}**

   **return 0;**

**}**

This program creates a hard link and prints the success message if appropriate. If any error occurs, it prints a failure message.

Note

The unlinking of both the hard and soft links can be done in the same command (i.e., using either the rm command or the unlink command. Programmatically, the same unlink system call which is available in the unistd.h unlink the link that is created. Once the link is unlinked, it is deleted from the system.

## System Calls for Directories

So far, various file concepts and their respective system calls have been covered. Several activities were performed programmatically. Now it’s time to dig deeper into the directories and various system functions that are associated with it. Directories are frequently used in daily life to organize files in a well-structured manner. This section discussed the various system calls that are available in C programming. The following are the most common operations performed in a directory.

- Creating a directory
    
- Removing a directory
    
- Getting the current working directory
    
- Changing a directory
    
- Reading a directory
    
- Closing a directory
    

### Creating a Directory

The creation of a directory is done with the mkdir function , which is available in the sys/stat.h library. The return type of this function is an integer. It returns 0 on the successful creation of a directory; it returns –1 for a failure.

The following shows the syntax.

**int mkdir(const char *path, mode_t mode);**

- **p**ath is the first argument that describes the path and the new directory name to create in the system.
    
- mode represents the permissions to give to a new directory.
    

Let’s look at a simple example of creating a directory programmatically. The following code creates a new directory in the current program location.

**#include<stdio.h>**

**#include<sys/stat.h>**

**#include<sys/types.h>**

**int main(){**

   **int isCreated;**

   **char *DIR_NAME;**

   **printf("Enter the Directory name you want to create: ");**

   **scanf("%[^\n]%*c", DIR_NAME);**

 **// You can Set your own permissions based on your Requirements.**

   **isCreated = mkdir(DIR_NAME, 0777);**

   **if(isCreated == 0){ // The value is 0 for Successful**

       **printf("Directory is Created Successfully\n");**

   **}else{  // Value is -1 if it is unsuccessful.**

       **printf("Unable to Create Directory\n");**

   **}**

   **return 0;**

**}**

This creates a new directory and prints a success message if the creation operation is successful; otherwise, it prints an error message.

### Deleting a Directory

The deletion of a directory is done with the rmdir function, which is available in the sys/stat.h library. The return type of this function is an integer. It returns 0 on the successful deletion of a directory; it returns –1 if a failure.

The following shows the syntax.

**int rmdir(const char *pathname);**

- pathname determines the directory name with the absolute path to remove from the system.
    

Let’s look at an example of how to delete a directory from the system.

**#include<stdio.h>**

**#include<sys/stat.h>**

**#include<sys/types.h>**

**int rmdir(char *dirname);**

**int main(){**

   **int isRemoved;**

   **char DIR_NAME[512];**

   **printf("Enter the Directory name you want to create: ");**

   **scanf("%[^\n]%*c", DIR_NAME);**

   **isRemoved = rmdir(DIR_NAME);**

   **if(isRemoved == 0){ // The value is 0 for Successful**

       **printf("Directory is Deleted Successfully\n");**

   **}else{  // Value is -1 if it is unsuccessful.**

       **printf("Unable to Delete Directory\n");**

   **}**

   **return 0;**

**}**

This program deletes the directory from the system and prints a success message for a successful operation; otherwise, it returns a failure message.

### Getting the Current Working Directory

The getcwd function gets the current working directory. It is available in the unistd.h library. The return of this function is a character data type. It returns the program’s current working directory.

The following shows the syntax.

**char getcwd(char *buffer, size_t buffersize);**

- buffer is the first argument; it describes the char array that stores the buffer content.
    
- buffersize is the second argument; it is the length of the buffer.
    

Let’s look at a program to print the current working directory using the C program.

**#include<stdio.h>**

**#include<unistd.h>**

**int main(){**

     **char DIR[75];**

     **printf("Current Working Directory is: %s\n", getcwd(DIR, 75));**

     **return 0;**

**}**

This program prints the current working directory when a success; otherwise, it prints NULL.

### Changing Directory

There is a chdir system call that changes directory in your operating system. It is available in the unistd.h library. The return type of this function is an integer. It returns 0 on the successful change in a directory; it returns –1 for a failure.

The following shows the syntax.

**int chdir(const char *path);**

- path describes the path to change.
    

Here is a C program that changes the directory in a system.

**#include<stdio.h>**

**#include<unistd.h>**

**int main(){**

**char DIR[75];**

**// Prints the Current Working Directory Before change**

**printf("Working Directory Before Operation:%s\n",**

**getcwd(DIR, 75));**

   **//The below chdir("..") change to the parent directory** **.**

   **int status = chdir("..");**

   **//Success ---> 0 & Failure ---> -1**

   **if(status == 0){**

       **printf("Directory Changed Successfully.!\n");**

   **}else{**

       **printf("Unable to change the Directory.\n");**

   **}**

**// Prints the Current Working Directory After change**

**printf("Working Directory After Operation: %s\n",**

**getcwd(DIR, 75));**

   **return 0;**

**}**

This program changes the working directory of the calling process.

### Reading a Directory

Two types of functions read the content in directories: opendir and readdir. They are available in the dirent.h library. The return type of the opendir function is the directory stream.

A directory stream is an ordered sequence of all directory entries in a directory. A directory entry represents the files. This directory stream points to the start position.

The return type of readdir is a dirent structure, which returns NULL if the directory reaches its end. Dirent is a built-in structure that is implemented in the dirent.h library.

The following shows the syntax.

**DIR *opendir(const char *path);**

- The path argument indicates the value that you want to open.
    

struct dirent *readdir(DIR *directorypointer);

- The **directorypointer** argument should contain the directory stream pointer, which is a return value of the opendir function.
    

The internal structure of dirent is as follows.

**struct dirent{**

   **ino_t          d_ino;       // inode number**

   **off_t          d_off;       // offset to the next dirent**

   **unsigned short d_reclen;    // length of this record**

   **unsigned char  d_type;      // type of file;**

   **char           d_name[256]; // filename**

**};**

Let’s create a program in C that reads all the files in a directory and prints them to the console.

**#include <stdio.h>**

**#include<stdlib.h>**

**#include <dirent.h>**

 **int main() {**

   **// Directory Entry**

   **struct dirent *DIR_ENTRY;**

    **// opendir() returns a pointer of DIR type.**

   **DIR *DIR_READER = opendir(".");**

    **if (DIR_READER == NULL) {**

       **printf("Could not open current directory" );**

       **exit(1);**

   **}**

    **while ((DIR_ENTRY = readdir(DIR_READER)) != NULL)**

           **printf("%s\n", DIR_ENTRY->d_name);**

    **closedir(DIR_READER);**

   **return 0;**

**}**

This program returns all the files and folders present in the given location.

### Closing a Directory

The closedir function closes the directory stream that is running in a process. The return type of this function is an integer. It returns 0 on the successful closing of a directory; it returns –1 for a failure.

The following shows the syntax.

**int closedir(DIR *directorypointer);**

- directorypointer is an argument that contains the directory stream pointer, which is simply a return value of the opendir function.
    

Here’s an example.

**#include <stdio.h>**

**#include<stdlib.h>**

**#include <dirent.h>**

 **int main() {**

   **// Directory Entry**

   **struct dirent *DIR_ENTRY;**

    **// opendir() returns a pointer of DIR type.**

   **DIR *DIR_READER = opendir(".");**

    **if (DIR_READER == NULL) {**

       **printf("Could not open current directory" );**

       **exit(1);**

   **}**

   **int status = closedir(DIR_READER);**

   **if(status == 0){**

       **printf("Directory Closed Successfully.!");**

   **}else{**

       **printf("Unable to close the Directory.");**

   **}**

   **return 0;**

**}**

This program prints the success statement if the directory is closed successfully; otherwise, it prints a failure message.

## Summary

This chapter focused on the Unix file structure, including the following topics.

- A file’s metadata and inode structure and how the Unix system identifies a file in the system.
    
- The system calls that are available for file operations in the Unix system and various file I/O operations.
    
- How chmod() changes file permissions programmatically. The access function checks file permissions with attributes like R_OK, W_OK, F_OK, and X_OK.
    
- The various Linux commands to create soft and hard links, including the programmatic ways to create them. This included a discussion on unlinking a link in both command-based and programmatic ways.
    
- The various directory system calls that manipulate directory operations.
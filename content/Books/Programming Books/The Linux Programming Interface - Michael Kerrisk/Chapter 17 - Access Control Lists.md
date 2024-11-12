---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 17 - Access Control Lists
modified: 2024-11-11T19:16:50-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **17**  
**ACCESS CONTROL LISTS**

[Section 15.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#ch15lev1sec04) described the traditional UNIX (and Linux) file permissions scheme. For many applications, this scheme is sufficient. However, some applications need finer control over the permissions granted to specific users and groups. To meet this requirement, many UNIX systems implement an extension to the traditional UNIX file permissions model known as _access control lists_ (ACLs). ACLs allow file permissions to be specified per user or per group, for an arbitrary number of users and groups. Linux provides ACLs from kernel 2.6 onward.

Support for ACLs is optional for each file system, and is controlled by kernel configuration options under the _File systems_ menu. _Reiserfs_ support for ACLs has been available since kernel 2.6.7.

In order to be able to create ACLs on an _ext2_, _ext3_, _ext4_, or _Reiserfs_ file system, the file system must be mounted with the _mount –o acl_ option.

ACLs have never been formally standardized for UNIX systems. An attempt was made to do this in the form of the POSIX.1e and POSIX.2c draft standards, which aimed to specify, respectively, the application program interface (API) and the shell commands for ACLs (as well as other features, such as capabilities). Ultimately, this standardization attempt foundered, and these draft standards were withdrawn. Nevertheless, many UNIX implementations (including Linux) base their ACL implementations on these draft standards (usually on the final version, _Draft 17_). However, because there are many variations across ACL implementations (in part springing from the incompleteness of the draft standards), writing portable programs that use ACLs presents some difficulties.

This chapter provides a description of ACLs and a brief tutorial on their use. It also describes some of the library functions used for manipulating and retrieving ACLs. We won’t go into detail on all of these functions because there are so many of them. (For the details, see the manual pages.)

### **17.1 Overview**

An ACL is a series of ACL entries, each of which defines the file permissions for an individual user or group of users (see [Figure 17-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17fig1)).

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f17-01.jpg)

**Figure 17-1:** An access control list

##### **ACL entries**

Each ACL entry consists of the following parts:

• a _tag type_, which indicates whether this entry applies to a user, to a group, or to some other category of user;

• an optional _tag qualifier_, which identifies a specific user or group (i.e., a user ID or a group ID); and

• a _permission set_, which specifies the permissions (read, write, and execute) that are granted by the entry.

The tag type has one of the following values:

ACL_USER_OBJ

This entry specifies the permissions granted to the file owner. Each ACL contains exactly one ACL_USER_OBJ entry. This entry corresponds to the traditional file _owner_ (_user_) permissions.

ACL_USER

This entry specifies the permissions granted to the user identified by the tag qualifier. An ACL may contain zero or more ACL_USER entries, but at most one ACL_USER entry may be defined for a particular user.

ACL_GROUP_OBJ

This entry specifies permissions granted to the file group. Each ACL contains exactly one ACL_GROUP_OBJ entry. This entry corresponds to the traditional file _group_ permissions, unless the ACL also contains an ACL_MASK entry.

ACL_GROUP

This entry specifies the permissions granted to the group identified by the tag qualifier. An ACL may contain zero or more ACL_GROUP entries, but at most one ACL_GROUP entry may be defined for a particular group.

ACL_MASK

This entry specifies the maximum permissions that may be granted by ACL_USER, ACL_GROUP_OBJ, and ACL_GROUP entries. An ACL contains at most one ACL_MASK entry. If the ACL contains ACL_USER or ACL_GROUP entries, then an ACL_MASK entry is mandatory. We say more about this tag type shortly.

ACL_OTHER

This entry specifies the permissions that are granted to users that don’t match any other ACL entry. Each ACL contains exactly one ACL_OTHER entry. This entry corresponds to the traditional file _other_ permissions.

The tag qualifier is employed only for ACL_USER and ACL_GROUP entries. It specifies either a user ID or a group ID.

##### **Minimal and extended ACLs**

A _minimal_ ACL is one that is semantically equivalent to the traditional file permission set. It contains exactly three entries: one of each of the types ACL_USER_OBJ, ACL_GROUP_OBJ, and ACL_OTHER. An _extended_ ACL is one that additionally contains ACL_USER, ACL_GROUP, and ACL_MASK entries.

One reason for drawing a distinction between minimal ACLs and extended ACLs is that the latter provide a semantic extension to the traditional permissions model. Another reason concerns the Linux implementation of ACLs. ACLs are implemented as _system_ extended attributes ([Chapter 16](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch16.xhtml#ch16)). The extended attribute used for maintaining a file access ACL is named _system.posix_acl_access_. This extended attribute is required only if the file has an extended ACL. The permissions information for a minimal ACL can be (and is) stored in the traditional file permission bits.

### **17.2 ACL Permission-Checking Algorithm**

Permission checking on a file that has an ACL is performed in the same circumstances as for the traditional file permissions model ([Section 15.4.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#ch15lev2sec07)). Checks are performed in the following order, until one of the criteria is matched:

1. If the process is privileged, all access is granted. There is one exception to this statement, analogous to the traditional permissions model described in [Section 15.4.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#ch15lev2sec07). When executing a file, a privileged process is granted execute permission only if that permission is granted via at least one of the ACL entries on the file.
    
2. If the effective user ID of the process matches the owner (user ID) of the file, then the process is granted the permissions specified in the ACL_USER_OBJ entry. (To be strictly accurate, on Linux, it is the process’s file-system IDs, rather than its effective IDs, that are used for the checks described in this section, as described in [Section 9.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch09.xhtml#ch09lev1sec05).)
    
3. If the effective user ID of the process matches the tag qualifier in one of the ACL_USER entries, then the process is granted the permissions specified in that entry, masked (ANDed) against the value of the ACL_MASK entry.
    
4. If one of the process’s group IDs (i.e., the effective group ID or any of the supplementary group IDs) matches the file group (this corresponds to the ACL_GROUP_OBJ entry) or the tag qualifier of any of the ACL_GROUP entries, then access is determined by checking each of the following, until a match is found:
    
    a) If one of the process’s group IDs matches the file group, and the ACL_GROUP_OBJ entry grants the requested permissions, then this entry determines the access granted to the file. The granted access is restricted by masking (ANDing) against the value in the ACL_MASK entry, if present.
    
    b) If one of the process’s group IDs matches the tag qualifier in an ACL_GROUP entry for the file, and that entry grants the requested permissions, then this entry determines the permissions granted. The granted access is restricted by masking (ANDing) against the value in the ACL_MASK entry.
    
    c) Otherwise, access is denied.
    
5. Otherwise, the process is granted the permissions specified in the ACL_OTHER entry.
    

We can clarify the rules relating to group IDs with some examples. Suppose we have a file whose group ID is 100, and that file is protected by the ACL shown in [Figure 17-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17fig1). If a process whose group ID is 100 makes the call _access(file, R_OK)_, then that call would succeed (i.e., return 0). (We describe _access()_ in [Section 15.4.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#ch15lev2sec08).) On the other hand, even though the ACL_GROUP_OBJ entry grants all permissions, the call _access(file, R_OK | W_OK | X_OK)_ would fail (i.e., return –1, with _errno_ set to EACCES) because the ACL_GROUP_OBJ permissions are masked (ANDed) against the ACL_MASK entry, and this entry denies execute permission.

As another example using [Figure 17-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17fig1), suppose we have a process that has a group ID of 102 and that also contains the group ID 103 in its supplementary group IDs. For this process, the calls _access(file, R_OK)_ and _access(file, W_OK)_ would both succeed, since they would match the ACL_GROUP entries for the group IDs 102 and 103, respectively. On the other hand, the call _access(file, R_OK | W_OK)_ would fail because there is no matching ACL_GROUP entry that grants both read and write permissions.

### **17.3 Long and Short Text Forms for ACLs**

When manipulating ACLs using the _setfacl_ and _getfacl_ commands (described in a moment) or certain ACL library functions, we specify textual representations of the ACL entries. Two formats are permitted for these textual representations:

• _Long text form_ ACLs contain one ACL entry per line, and may include comments, which are started by a # character and continue to the end-of-line. The _getfacl_ command displays ACLs in long text form. The _setfacl –M acl-file_ option, which takes an ACL specification from a file, expects the specification to be in long text form.

• _Short text form_ ACLs consist of a sequence of ACL entries separated by commas.

In both forms, each ACL entry consists of three parts separated by colons:

_tag-type_:[_tag-qualifier_]: _permissions_

The _tag-type_ is one of the values shown in the first column of [Table 17-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17table1). The _tag-type_ may optionally be followed by a _tag-qualifier_, which identifies a user or group, either by name or numeric identifier. The _tag-qualifier_ is present only for ACL_USER and ACL_GROUP entries.

The following are all short text form ACLs corresponding to a traditional permissions mask of 0650:

u::rw-,g::r-x,o::---  
u::rw,g::rx,o::-  
user::rw,group::rx,other::-

The following short text form ACL includes two named users, a named group, and a mask entry:

u::rw,u:paulh:rw,u:annabel:rw,g::r,g:teach:rw,m::rwx,o::-

**Table 17-1:** Interpretation of ACL entry text forms

|**Tag text forms**|**Tag qualifier present?**|**Corresponding tag type**|**Entry for**|
|---|---|---|---|
|u, user|N|ACL_USER_OBJ|File owner (user)|
|u, user|Y|ACL_USER|Specified user|
|g, group|N|ACL_GROUP_OBJ|File group|
|g, group|Y|ACL_GROUP|Specified group|
|m, mask|N|ACL_MASK|Mask for group class|
|o, other|N|ACL_OTHER|Other users|

### **17.4 The** ACL_MASK **Entry and the ACL Group Class**

If an ACL contains ACL_USER or ACL_GROUP entries, then it must contain an ACL_MASK entry. If the ACL doesn’t contain any ACL_USER or ACL_GROUP entries, then the ACL_MASK entry is optional.

The ACL_MASK entry acts as an upper limit on the permissions granted by ACL entries in the so-called _group class_. The group class is the set of all ACL_USER, ACL_GROUP, and ACL_GROUP_OBJ entries in the ACL.

The purpose of the ACL_MASK entry is to provide consistent behavior when running ACL-unaware applications. As an example of why the mask entry is needed, suppose that the ACL on a file includes the following entries:

user::rwx                     # ACL_USER_OBJ  
user:paulh:r-x                # ACL_USER  
group::r-x                    # ACL_GROUP_OBJ  
group:teach:--x               # ACL_GROUP  
other::--x                    # ACL_OTHER

Now suppose that a program executes the following _chmod()_ call on this file:

chmod(pathname, 0700);     /* Set permissions to rwx------ */

In an ACL-unaware application, this means “Deny access to everyone except the file owner.” These semantics should hold even in the presence of ACLs. In the absence of an ACL_MASK entry, this behavior could be implemented in various ways, but there are problems with each approach:

• Simply modifying the ACL_GROUP_OBJ and ACL_OTHER entries to have the mask ---would be insufficient, since the user _paulh_ and the group _teach_ would still have some permissions on the file.

• Another possibility would be to apply the new group and other permission settings (i.e., all permissions disabled) to all of the ACL_USER, ACL_GROUP, ACL_GROUP_OBJ, and ACL_OTHER entries in the ACL:

user::rwx                     # ACL_USER_OBJ  
user:paulh:---                # ACL_USER  
group::---                    # ACL_GROUP_OBJ  
group:teach:---               # ACL_GROUP  
other::---                    # ACL_OTHER

The problem with this approach is that the ACL-unaware application would thereby inadvertently destroy the file permission semantics established by ACL-aware applications, since the following call (for example) would not restore the ACL_USER and ACL_GROUP entries of the ACL to their former states:

chmod(pathname, 0751);

• To avoid these problems, we might consider making the ACL_GROUP_OBJ entry the limiting set for all ACL_USER and ACL_GROUP entries. However, this would mean that the ACL_GROUP_OBJ permissions would always need to be set to the union of all permissions allowed in all ACL_USER and ACL_GROUP entries. This would conflict with the use of the ACL_GROUP_OBJ entry for determining the permissions accorded to the file group.

The ACL_MASK entry was devised to solve these problems. It provides a mechanism that allows the traditional meanings of _chmod()_ operations to be implemented, without destroying the file permission semantics established by ACL-aware applications. When an ACL has an ACL_MASK entry:

• all changes to traditional group permissions via _chmod()_ change the setting of the ACL_MASK entry (rather than the ACL_GROUP_OBJ entry); and

• a call to _stat()_ returns the ACL_MASK permissions (instead of the ACL_GROUP_OBJ permissions) in the group permission bits of the _st_mode_ field ([Figure 15-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#ch15fig1), on [page 281](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#page_281)).

While the ACL_MASK entry provides a way of preserving ACL information in the face of ACL-unaware applications, the reverse is not guaranteed. The presence of ACLs overrides the effect of traditional operations on file group permissions. For example, suppose that we have placed the following ACL on a file:

user::rw-,group::---,mask::---,other::r--

If we then execute the command _chmod g+rw_ on this file, the ACL becomes:

user::rw-,group::---,mask::rw-,other::r--

In this case, group still has no access to the file. One workaround for this is to modify the ACL entry for group to grant all permissions. Consequently, group will then always obtain whatever permissions are granted to the ACL_MASK entry.

### **17.5 The _getfacl_ and _setfacl_ Commands**

From the shell, we can use the _getfacl_ command to view the ACL on a file.

$ umask 022                     Set shell umask to known state  
$ touch tfile                   Create a new file  
$ getfacl tfile  
# file: tfile  
# owner: mtk  
# group: users  
user::rw-  
group::r--  
other::r--

From the output of the _getfacl_ command, we see that the new file is created with a minimal ACL. When displaying the text form of this ACL, _getfacl_ precedes the ACL entries with three lines showing the name and ownership of the file. We can prevent these lines from being displayed by specifying the _––omit–header_ option.

Next, we demonstrate that changes to a file’s permissions using the traditional _chmod_ command are carried through to the ACL.

$ chmod u=rwx,g=rx,o=x tfile  
$ getfacl --omit-header tfile  
user::rwx  
group::r-x  
other::--x

The _setfacl_ command modifies a file ACL. Here, we use the _setfacl –m_ command to add an ACL_USER and an ACL_GROUP entry to the ACL:

$ setfacl -m u:paulh:rx,g:teach:x tfile  
$ getfacl --omit-header tfile  
user::rwx  
user:paulh:r-x                      ACL_USER entry  
group::r-x  
group:teach:--x                     ACL_GROUP entry  
mask::r-x                           ACL_MASK entry  
other::--x

The _setfacl –m_ option modifies existing ACL entries, or adds new entries if corresponding entries with the given tag type and qualifier do not already exist. We can additionally use the _–R_ option to recursively apply the specified ACL to all of the files in a directory tree.

From the output of the _getfacl_ command, we can see that _setfacl_ automatically created an ACL_MASK entry for this ACL.

The addition of the ACL_USER and ACL_GROUP entries converts this ACL into an extended ACL, and _ls –l_ indicates this fact by appending a plus sign (+) after the traditional file permissions mask:

$ ls -l tfile  
-rwxr-x--x+   1 mtk     users         0 Dec 3 15:42 tfile

We continue by using _setfacl_ to disable all permissions except execute on the ACL_MASK entry, and then view the ACL once more with _getfacl_:

$ setfacl -m m::x tfile  
$ getfacl --omit-header tfile  
user::rwx  
user:paulh:r-x                 #effective:--x  
group::r-x                     #effective:--x  
group:teach:--x  
mask::--x  
other::--x

The #effective: comments that _getfacl_ prints after the entries for the user _paulh_ and the file group (group::) inform us that after masking (ANDing) against the ACL_MASK entry, the permissions granted by each of these entries will actually be less than those specified in the entry.

We then use _ls –l_ to once more view the traditional permission bits of the file. We see that the displayed group class permission bits reflect the permissions in the ACL_MASK entry (--x), rather than those in the ACL_GROUP entry (r-x):

$ ls -l tfile  
-rwx--x--x+   1 mtk     users           0 Dec 3 15:42 tfile

The _setfacl –x_ command can be used to remove entries from an ACL. Here, we remove the entries for the user _paulh_ and the group _teach_ (no permissions are specified when removing entries):

$ setfacl -x u:paulh,g:teach tfile  
$ getfacl --omit-header tfile  
user::rwx  
group::r-x  
mask::r-x  
other::--x

Note that during the above operation, _setfacl_ automatically adjusted the mask entry to be the union of all of the group class entries. (There was just one such entry: ACL_GROUP_OBJ.) If we want to prevent such adjustment, then we must specify the _–n_ option to _setfacl_.

Finally, we note that the _setfacl –b_ option can be used to remove all extended entries from an ACL, leaving just the minimal (i.e., user, group, and other) entries.

### **17.6 Default ACLs and File Creation**

In the discussion of ACLs so far, we have been describing _access_ ACLs. As its name implies, an access ACL is used in determining the permissions that a process has when accessing the file associated with the ACL. We can create a second type of ACL on directories: a _default_ ACL.

A default ACL plays no part in determining the permissions granted when accessing the directory. Instead, its presence or absence determines the ACL(s) and permissions that are placed on files and subdirectories that are created in the directory. (A default ACL is stored as an extended attribute named _system.posix_acl_default_.)

To view and set the default ACL of a directory, we use the _–d_ option of the _getfacl_ and _setfacl_ commands.

$ mkdir sub  
$ setfacl -d -m u::rwx,u:paulh:rx,g::rx,g:teach:rwx,o::- sub  
$ getfacl -d --omit-header sub  
user::rwx  
user:paulh:r-x  
group::r-x  
group:teach:rwx  
mask::rwx                       setfacl generated ACL_MASK entry automatically  
other::---

We can remove a default ACL from a directory using the _setfacl –k_ option.

If a directory has a default ACL, then:

• A new subdirectory created in this directory inherits the directory’s default ACL as its default ACL. In other words, default ACLs propagate down through a directory tree as new subdirectories are created.

• A new file or subdirectory created in this directory inherits the directory’s default ACL as its access ACL. The ACL entries that correspond to the traditional file permission bits are masked (ANDed) against the corresponding bits of the _mode_ argument in the system call (_open()_, _mkdir()_, and so on) used to create the file or subdirectory. By “corresponding ACL entries,” we mean:

– ACL_USER_OBJ;

– ACL_MASK or, if ACL_MASK is absent, then ACL_GROUP_OBJ; and

– ACL_OTHER.

When a directory has a default ACL, the process umask ([Section 15.4.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#ch15lev2sec10)) doesn’t play a part in determining the permissions in the entries of the access ACL of a new file created in that directory.

As an example of how a new file inherits its access ACL from the parent directory’s default ACL, suppose we used the following _open()_ call to create a new file in the directory created above:

open("sub/tfile", O_RDWR | O_CREAT,  
        S_IRWXU | S_IXGRP | S_IXOTH);   /* rwx--x--x */

The new file would have the following access ACL:

$ getfacl --omit-header sub/tfile  
user::rwx  
user:paulh:r-x                  #effective:--x  
group::r-x                      #effective:--x  
group:teach:rwx                 #effective:--x  
mask::--x  
other::---

If a directory doesn’t have a default ACL, then:

• New subdirectories created in this directory also do not have a default ACL.

• The permissions of the new file or directory are set following the traditional rules ([Section 15.4.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch15.xhtml#ch15lev2sec10)): the file permissions are set to the value in the _mode_ argument (given to _open()_, _mkdir()_, and so on), less the bits that are turned off by the process umask. This results in a minimal ACL on the new file.

### **17.7 ACL Implementation Limits**

The various file-system implementations impose limits on the number of entries in an ACL:

• On _ext2_, _ext3_, and _ext4_, the total number of ACLs on a file is governed by the requirement that the bytes in all of the names and values of a file’s extended attributes must be contained in a single logical disk block ([Section 16.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch16.xhtml#ch16lev1sec02)). Each ACL entry requires 8 bytes, so that the maximum number of ACL entries for a file is somewhat less (because of some overhead for the name of the extended attribute for the ACL) than one-eighth of the block size. Thus, a 4096-byte block size allows for a maximum of around 500 ACL entries. (Kernels before 2.6.11 imposed an arbitrary limitation of 32 entries for ACLs on _ext2_ and _ext3_.)

• On _XFS_, an ACL is limited to 25 entries.

• On _Reiserfs_ and _JFS_, ACLs can contain up to 8191 entries. This limit is a consequence of the size limitation (64 kB) imposed by the VFS on the value of an extended attribute ([Section 16.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch16.xhtml#ch16lev1sec02)).

At the time of writing, _Btrfs_ limits ACLs to around 500 entries. However, since _Btrfs_ was still under heavy development, this limit may change.

Although most of the above file systems allow large numbers of entries to be created in an ACL, this should be avoided for the following reasons:

• The maintenance of lengthy ACLs becomes a complex and potentially error-prone system administration task.

• The amount of time required to scan the ACL for the matching entry (or matching entries in the case of group ID checks) increases linearly with the number of ACL entries.

Generally, we can keep the number of ACL entries on a file down to a reasonable number by defining suitable groups in the system group file ([Section 8.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch08.xhtml#ch08lev1sec03)) and using those groups within the ACL.

### **17.8 The ACL API**

The POSIX.1e draft standard defined a large suite of functions and data structures for manipulating ACLs. Since they are so numerous, we won’t attempt to describe the details of all of these functions. Instead, we provide an overview of their usage and conclude with an example program.

Programs that use the ACL API should include <sys/acl.h>. It may also be necessary to include <acl/libacl.h> if the program makes use of various Linux extensions to the POSIX.1e draft standard. (A list of the Linux extensions is provided in the _acl(5)_ manual page.) Programs using this API must be compiled with the _–lacl_ option, in order to link against the _libacl_ library.

As already noted, on Linux, ACLs are implemented using extended attributes, and the ACL API is implemented as a set of library functions that manipulate user-space data structures, and, where necessary, make calls to _getxattr()_ and _setxattr()_ to retrieve and modify the on-disk _system_ extended attribute that holds the ACL representation. It is also possible (though not recommended) for an application to use _getxattr()_ and _setxattr()_ to manipulate ACLs directly.

##### **Overview**

The functions that constitute the ACL API are listed in the _acl(5)_ manual page. At first sight, this plethora of functions and data structures can seem bewildering. [Figure 17-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17fig2) provides an overview of the relationship between the various data structures and indicates the use of many of the ACL functions.

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f17-02.jpg)

**Figure 17-2:** Relationship between ACL library functions and data structures

From [Figure 17-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17fig2), we can see that the ACL API considers an ACL as a hierarchical object:

• An ACL consists of one or more ACL entries.

• Each ACL entry consists of a tag type, an optional tag qualifier, and a permission set.

We now look briefly at the various ACL functions. In most cases, we don’t describe the error returns from each function. Functions that return an integer (_status_) typically return 0 on success and –1 on error. Functions that return a handle (pointer) return NULL on error. Errors can be diagnosed using _errno_ in the usual manner.

A _handle_ is an abstract term for some technique used to refer to an object or data structure. The representation of a handle is private to the API implementation. It may be, for example, a pointer, an array index, or a hash key.

##### **Fetching a file’s ACL into memory**

The _acl_get_file()_ function retrieves a copy of the ACL of the file identified by _pathname_.

acl_t acl;  
  
acl = acl_get_file(pathname, type);

This function retrieves either the access ACL or the default ACL, depending on whether _type_ is specified as ACL_TYPE_ACCESS or ACL_TYPE_DEFAULT. As its function result, _acl_get_file()_ returns a handle (of type _acl_t_) for use with other ACL functions.

##### **Retrieving entries from an in-memory ACL**

The _acl_get_entry()_ function returns a handle (of type _acl_entry_t_) referring to one of the ACL entries within the in-memory ACL referred to by its _acl_ argument. This handle is returned in the location pointed to by the final function argument.

acl_entry_t entry;  
  
status = acl_get_entry(acl, entry_id, &entry);

The _entry_id_ argument determines which entry’s handle is returned. If _entry_id_ is specified as ACL_FIRST_ENTRY, then a handle for the first entry in the ACL is returned. If _entry_id_ is specified as ACL_NEXT_ENTRY, then a handle is returned for the entry following the last ACL entry that was retrieved. Thus, we can loop through all of the entries in an ACL by specifying _entry_id_ as ACL_FIRST_ENTRY in the first call to _acl_get_entry()_ and specifying _entry_id_ as ACL_NEXT_ENTRY in subsequent calls.

The _acl_get_entry()_ function returns 1 if it successfully fetches an ACL entry, 0 if there are no more entries, or –1 on error.

##### **Retrieving and modifying attributes in an ACL entry**

The _acl_get_tag_type()_ and _acl_set_tag_type()_ functions retrieve and modify the tag type in the ACL entry referred to by their _entry_ argument.

acl_tag_t tag_type;  
  
status = acl_get_tag_type(entry, &tag_type);  
status = acl_set_tag_type(entry, tag_type);

The _tag_type_ argument has the type _acl_tag_t_ (an integer type), and has one of the values ACL_USER_OBJ, ACL_USER, ACL_GROUP_OBJ, ACL_GROUP, ACL_OTHER, or ACL_MASK.

The _acl_get_qualifier()_ and _acl_set_qualifier()_ functions retrieve and modify the tag qualifier in the ACL entry referred to by their _entry_ argument. Here is an example, in which we assume that we have already determined that this is an ACL_USER entry by inspecting the tag type:

uid_t *qualp;               /* Pointer to UID */  
  
qualp = acl_get_qualifier(entry);  
status = acl_set_qualifier(entry, qualp);

The tag qualifier is valid only if the tag type of this entry is ACL_USER or ACL_GROUP. In the former case, _qualp_ is a pointer to a user ID (_uid_t *_); in the latter case, it is a pointer to a group ID (_gid_t *_).

The _acl_get_permset()_ and _acl_set_permset()_ functions retrieve and modify the permission set in the ACL entry referred to by their _entry_ argument.

acl_permset_t permset;  
  
status = acl_get_permset(entry, &permset);  
status = acl_set_permset(entry, permset);

The _acl_permset_t_ data type is a handle referring to a permission set.

The following functions are used to manipulate the contents of a permission set:

int is_set;  
  
is_set = acl_get_perm(permset, perm);  
  
status = acl_add_perm(permset, perm);  
status = acl_delete_perm(permset, perm);  
status = acl_clear_perms(permset);

In each of these calls, _perm_ is specified as ACL_READ, ACL_WRITE, or ACL_EXECUTE, with the obvious meanings. These functions are used as follows:

• The _acl_get_perm()_ function returns 1 (true) if the permission specified in _perm_ is enabled in the permission set referred to by _permset_, or 0 if it is not. This function is a Linux extension to the POSIX.1e draft standard.

• The _acl_add_perm()_ function adds the permission specified in _perm_ to the permission set referred to by _permset_.

• The _acl_delete_perm()_ function removes the permission specified in _perm_ from the permission set referred to by _permset_. (It is not an error to remove a permission if it is not present in the set.)

• The _acl_clear_perms()_ function removes all permissions from the permission set referred to by _permset_.

##### **Creating and deleting ACL entries**

The _acl_create_entry()_ function creates a new entry in an existing ACL. A handle referring to the new entry is returned in the location pointed to by the second function argument.

acl_entry_t entry;  
  
status = acl_create_entry(&acl, &entry);

The new entry can then be populated using the functions described previously.

The _acl_delete_entry()_ function removes an entry from an ACL.

status = acl_delete_entry(acl, entry);

##### **Updating a file’s ACL**

The _acl_set_file()_ function is the converse of _acl_get_file()_. It updates the on-disk ACL with the contents of the in-memory ACL referred to by its _acl_ argument.

int status;  
  
status = acl_set_file(pathname, type, acl);

The _type_ argument is either ACL_TYPE_ACCESS, to update the access ACL, or ACL_TYPE_DEFAULT, to update a directory’s default ACL.

##### **Converting an ACL between in-memory and text form**

The _acl_from_text()_ function translates a string containing a long or short text form ACL into an in-memory ACL, and returns a handle that can be used to refer to the ACL in subsequent function calls.

acl = acl_from_text(acl_string);

The _acl_to_text()_ function performs the reverse conversion, returning a long text form string corresponding to the ACL referred to by its _acl_ argument.

char *str;  
ssize_t len;  
  
str = acl_to_text(acl, &len);

If the _len_ argument is not specified as NULL, then the buffer it points to is used to return the length of the string returned as the function result.

##### **Other functions in the ACL API**

The following paragraphs describe several other commonly used ACL functions that are not shown in [Figure 17-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17fig2).

The _acl_calc_mask(&acl)_ function calculates and sets the permissions in the ACL_MASK entry of the in-memory ACL whose handle is pointed to by its argument. Typically, we use this function whenever we create or modify an ACL. The ACL_MASK permissions are calculated as the union of the permissions in all ACL_USER, ACL_GROUP, and ACL_GROUP_OBJ entries. A useful property of this function is that it creates the ACL_MASK entry if it doesn’t already exist. This means that if we add ACL_USER and ACL_GROUP entries to a previously minimal ACL, then we can use this function to ensure the creation of the ACL_MASK entry.

The _acl_valid(acl)_ function returns 0 if the ACL referred to by its argument is valid, or –1 otherwise. An ACL is valid if all of the following are true:

• the ACL_USER_OBJ, ACL_GROUP_OBJ, and ACL_OTHER entries appear exactly once;

• there is an ACL_MASK entry if any ACL_USER or ACL_GROUP entries are present;

• there is at most one ACL_MASK entry;

• each ACL_USER entry has a unique user ID; and

• each ACL_GROUP entry has a unique group ID.

The _acl_check()_ and _acl_error()_ functions (both are Linux extensions) are alternatives to _acl_valid()_ that are less portable, but provide a more precise description of the error in a malformed ACL. See the manual pages for details.

The _acl_delete_def_file(pathname)_ function removes the default ACL on the directory referred to by _pathname_.

The _acl_init(count)_ function creates a new, empty ACL structure that initially contains space for at least _count_ ACL entries. (The _count_ argument is a hint to the system about intended usage, not a hard limit.) A handle for the new ACL is returned as the function result.

The _acl_dup(acl)_ function creates a duplicate of the ACL referred to by _acl_ and returns a handle for the duplicate ACL as its function result.

The _acl_free(handle)_ function frees memory allocated by other ACL functions. For example, we must use _acl_free()_ to free memory allocated by calls to _acl_from_text()_, _acl_to_text()_, _acl_get_file()_, _acl_init()_, and _acl_dup()_.

##### **Example program**

[Listing 17-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17ex1) demonstrates the use of some of the ACL library functions. This program retrieves and displays the ACL on a file (i.e., it provides a subset of the functionality of the _getfacl_ command). If the _–d_ command-line option is specified, then the program displays the default ACL (of a directory) instead of the access ACL.

Here is an example of the use of this program:

$ touch tfile  
$ setfacl -m 'u:annie:r,u:paulh:rw,g:teach:r' tfile  
$ ./acl_view tfile  
user_obj             rw-  
user        annie    r--  
user        paulh    rw-  
group_obj            r--  
group       teach    r--  
mask                 rw-  
other                r--

The source code distribution of this book also includes a program, acl/acl_update.c, that performs updates on an ACL (i.e., it provides a subset of the functionality of the _setfacl_ command).

**Listing 17-1:** Display the access or default ACL on a file

___________________________________________________________ acl/acl_view.c  
  
#include <acl/libacl.h>  
#include <sys/acl.h>  
#include "ugid_functions.h"  
#include "tlpi_hdr.h"  
  
static void  
usageError(char *progName)  
{  
    fprintf(stderr, "Usage: %s [-d] filename\n", progName);  
    exit(EXIT_FAILURE);  
}  
  
int  
main(int argc, char *argv[])  
{  
    acl_t acl;  
    acl_type_t type;  
    acl_entry_t entry;  
    acl_tag_t tag;  
    uid_t *uidp;  
    gid_t *gidp;  
    acl_permset_t permset;  
    char *name;  
    int entryId, permVal, opt;  
  
    type = ACL_TYPE_ACCESS;  
    while ((opt = getopt(argc, argv, "d")) != -1) {  
        switch (opt) {  
        case 'd': type = ACL_TYPE_DEFAULT;      break;  
        case '?': usageError(argv[0]);  
        }  
    }  
  
    if (optind + 1 != argc)  
        usageError(argv[0]);  
  
    acl = acl_get_file(argv[optind], type);  
    if (acl == NULL)  
        errExit("acl_get_file");  
  
    /* Walk through each entry in this ACL */  
  
    for (entryId = ACL_FIRST_ENTRY; ; entryId = ACL_NEXT_ENTRY) {  
  
        if (acl_get_entry(acl, entryId, &entry) != 1)  
            break;                      /* Exit on error or no more entries */  
  
        /* Retrieve and display tag type */  
  
        if (acl_get_tag_type(entry, &tag) == -1)  
            errExit("acl_get_tag_type");  
  
        printf("%-12s", (tag == ACL_USER_OBJ) ?  "user_obj" :  
                        (tag == ACL_USER) ?      "user" :  
                        (tag == ACL_GROUP_OBJ) ? "group_obj" :  
                        (tag == ACL_GROUP) ?     "group" :  
                        (tag == ACL_MASK) ?      "mask" :  
                        (tag == ACL_OTHER) ?     "other" : "???");  
  
        /* Retrieve and display optional tag qualifier */  
  
        if (tag == ACL_USER) {  
            uidp = acl_get_qualifier(entry);  
            if (uidp == NULL)  
                errExit("acl_get_qualifier");  
  
            name = userNameFromID(*uidp);  
            if (name == NULL)  
                printf("%-8d ", *uidp);  
            else  
                printf("%-8s ", name);  
  
            if (acl_free(uidp) == -1)  
                errExit("acl_free");  
  
        } else if (tag == ACL_GROUP) {  
            gidp = acl_get_qualifier(entry);  
            if (gidp == NULL)  
                errExit("acl_get_qualifier");  
  
            name = groupNameFromId(*gidp);  
            if (name == NULL)  
                printf("%-8d ", *gidp);  
            else  
                printf("%-8s ", name);  
  
            if (acl_free(gidp) == -1)  
                errExit("acl_free");  
  
        } else {  
            printf("         ");  
        }  
  
        /* Retrieve and display permissions */  
  
        if (acl_get_permset(entry, &permset) == -1)  
            errExit("acl_get_permset");  
  
        permVal = acl_get_perm(permset, ACL_READ);  
        if (permVal == -1)  
            errExit("acl_get_perm - ACL_READ");  
        printf("%c", (permVal == 1) ? 'r' : '-');  
        permVal = acl_get_perm(permset, ACL_WRITE);  
        if (permVal == -1)  
            errExit("acl_get_perm - ACL_WRITE");  
        printf("%c", (permVal == 1) ? 'w' : '-');  
        permVal = acl_get_perm(permset, ACL_EXECUTE);  
        if (permVal == -1)  
            errExit("acl_get_perm - ACL_EXECUTE");  
        printf("%c", (permVal == 1) ? 'x' : '-');  
  
        printf("\n");  
    }  
  
    if (acl_free(acl) == -1)  
        errExit("acl_free");  
  
    exit(EXIT_SUCCESS);  
}  
___________________________________________________________ acl/acl_view.c

### **17.9 Summary**

From version 2.6 onward, Linux supports ACLs. ACLs extend the traditional UNIX file permissions model, allowing file permissions to be controlled on a peruser and per-group basis.

##### **Further information**

The final versions (_Draft 17_) of the draft POSIX.1e and POSIX.2c standards are available online at _[http://wt.tuxomania.net/publications/posix.1e/](http://wt.tuxomania.net/publications/posix.1e/)_.

The _acl(5)_ manual page gives an overview of ACLs and some guidance on the portability of the various ACL library functions implemented on Linux.

Details of the Linux implementation of ACLs and extended attributes can be found in [[Grünbacher, 2003](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib38)]. Andreas Grünbacher maintains a web site containing information about ACLs at _[http://acl.bestbits.at/](http://acl.bestbits.at/)_.

### **17.10 Exercise**

**17-1.**   Write a program that displays the permissions from the ACL entry that corresponds to a particular user or group. The program should take two command-line arguments. The first argument is either of the letters _u_ or _g_, indicating whether the second argument identifies a user or group. (The functions defined in [Listing 8-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch08.xhtml#ch8ex1), on [page 159](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch08.xhtml#page_159), can be used to allow the second command-line argument to be specified numerically or as a name.) If the ACL entry that corresponds to the given user or group falls into the group class, then the program should additionally display the permissions that would apply after the ACL entry has been modified by the ACL mask entry.
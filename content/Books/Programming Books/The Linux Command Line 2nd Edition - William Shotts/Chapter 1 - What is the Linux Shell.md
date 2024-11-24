---
id: 01JCH5JREGTTWQC4GBRBD4M1KY
modified: 2024-11-12T16:52:32-05:00
title: Chapter 1 - What is the Linux Shell?
tags:
  - linux
  - terminal
  - shell
  - cli
  - books
---
## **1  
WHAT IS THE SHELL?**

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781492071235/files/images/common01.jpg)

When we speak of the command line, we are really referring to the _shell_. The shell is a program that takes keyboard commands and passes them to the operating system to carry out. Almost all Linux distributions supply a shell program from the GNU Project called bash. The name is an acronym for _b_ourne-_a_gain _sh_ell, a reference to the fact that bash is an enhanced replacement for sh, the original Unix shell program written by Steve Bourne.

### **Terminal Emulators**

When using a graphical user interface (GUI), we need another program called a _terminal emulator_ to interact with the shell. If we look through our desktop menus, we will probably find one. KDE uses konsole, and GNOME uses gnome-terminal, though it’s likely called simply Terminal on your menu. A number of other terminal emulators are available for Linux, but they all basically do the same thing: give us access to the shell. You will probably develop a preference for one or another terminal emulator based on the number of bells and whistles it has.

### **Making Your First Keystrokes**

So let’s get started. Launch the terminal emulator. Once it comes up, we should see something like this:

[me@linuxbox ~]$

This is called a _shell prompt_, and it will appear whenever the shell is ready to accept input. While it might vary in appearance somewhat depending on the distribution, it will typically include your _username@machinename_, followed by the current working directory (more about that in a little bit) and a dollar sign.

If the last character of the prompt is a hash mark (#) rather than a dollar sign, the terminal session has superuser privileges. This means either we are logged in as the root user or we selected a terminal emulator that provides superuser (administrative) privileges.

Assuming things are good so far, let’s try some typing. Enter some gibberish at the prompt like so:

[me@linuxbox ~]$ kaekfjaeifj

Because this command makes no sense, the shell tells us so and gives us another chance.

bash: kaekfjaeifj: command not found  
[me@linuxbox ~]$

#### **_Command History_**

If we press the up arrow, we will see that the previous command entered, kaekfjaeifj, reappears after the prompt. This is called _command history_. Most Linux distributions remember the last 1,000 commands by default. Press the down arrow and the previous command disappears.

#### **_Cursor Movement_**

Recall the previous command by pressing the up arrow again. If we try the left and right arrows, we’ll see that we can position the cursor anywhere on the command line. This makes editing commands easy.

**A FEW WORDS ABOUT MICE AND FOCUS**

While the shell is all about the keyboard, you can also use a mouse with your terminal emulator. A mechanism built into the X Window System (the underlying engine that makes the GUI go) supports a quick copy-and-paste technique. If you highlight some text by holding down the left mouse button and dragging the mouse over it (or double-clicking a word), it is copied into a buffer maintained by X. Pressing the middle mouse button will cause the text to be pasted at the cursor location. Try it.

Don’t be tempted to use CTRL-C and CTRL-V to perform copy and paste inside a terminal window. They don’t work. These control codes have different meanings to the shell and were assigned many years before the release of Microsoft Windows.

Your graphical desktop environment (most likely KDE or GNOME), in an effort to behave like Windows, probably has its focus policy set to “click to focus.” This means for a window to get focus (become active), you need to click on it. This is contrary to the traditional X behavior of “focus follows mouse,” which means that a window gets focus just by passing the mouse over it. The window will not come to the foreground until you click on it, but it will be able to receive input. Setting the focus policy to “focus follows mouse” will make the copy-and-paste technique even more useful. Give it a try if you can (some desktop environments such as Ubuntu’s Unity no longer support it). I think if you give it a chance, you will prefer it. You will find this setting in the configuration program for your window manager.

### **Try Some Simple Commands**

Now that we have learned to enter text in the terminal emulator, let’s try a few simple commands. Let’s begin with the date command, which displays the current time and date.

[me@linuxbox ~]$ date  
Fri Feb  2 15:09:41 EST 2018

A related command is cal, which, by default, displays a calendar of the current month.

[me@linuxbox ~]$ cal  
    February 2018     
Su Mo Tu We Th Fr Sa  
             1  2  3  
 4  5  6  7  8  9 10  
11 12 13 14 15 16 17  
18 19 20 21 22 23 24  
25 26 27 28      

**THE CONSOLE BEHIND THE CURTAIN**

Even if we have no terminal emulator running, several terminal sessions continue to run behind the graphical desktop. We can access these sessions, called _virtual consoles_, by pressing CTRL-ALT-F1 through CTRL-ALT-F6 on most Linux distributions. When a session is accessed, it presents a login prompt into which we can enter our username and password. To switch from one virtual console to another, press ALT-F1 through ALT-F6. On most systems, we can return to the graphical desktop by pressing ALT-F7.

To see the current amount of free space on our disk drives, enter df.

[me@linuxbox ~]$ df  
Filesystem           1K-blocks      Used Available Use% Mounted on  
/dev/sda2             15115452   5012392   9949716  34% /  
/dev/sda5             59631908  26545424  30008432  47% /home  
/dev/sda1               147764     17370    122765  13% /boot  
tmpfs                   256856         0    256856   0% /dev/shm

Likewise, to display the amount of free memory, enter the free command.

[me@linuxbox ~]$ free  
         total       used       free     shared    buffers     cached  
Mem:    513712     503976       9736          0       5312     122916  
-/+ buffers/cache: 375748     137964  
Swap:  1052248     104712     947536

### **Ending a Terminal Session**

We can end a terminal session by closing the terminal emulator window, by entering the exit command at the shell prompt, or by pressing CTRL-D.

[me@linuxbox ~]$ exit

### **Summing Up**

This chapter marked the beginning of our journey into the Linux command line, with an introduction to the shell, a glimpse of the command line, and a brief lesson on how to start and end a terminal session. We also saw how to issue some simple commands and perform a little light command-line editing. That wasn’t so scary, was it?

In the next chapter, we’ll learn a few more commands and wander around the Linux file system.

- Highlight
- Add Note
- Copy
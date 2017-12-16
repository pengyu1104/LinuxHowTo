# LinuxHowTo
Useful linux 
Linux audit files to see who made changes to a file
Posted on March 19, 2007in Categories File system, GNU/Open source, Howto, Linux, Monitoring, RedHat/Fedora Linux, Security, Sys admin, Tips last updated March 19, 2007

This is one of the key questions many new sys admin ask:



In order to use audit facility you need to use following utilities
=> auditctl – a command to assist controlling the kernel audit system. You can get status, and add or delete rules into kernel audit system. Setting a watch on a file is accomplished using this command:

=> ausearch – a command that can query the audit daemon logs based for events based on different search criteria.

=> aureport – a tool that produces summary reports of the audit system logs.

Note that following all instructions are tested on CentOS 4.x and Fedora Core and RHEL 4/5 Linux.

Task: install audit package
The audit package contains the user space utilities for storing and searching the audit records generate by the audit subsystem in the Linux 2.6 kernel. CentOS/Red Hat and Fedora core includes audit rpm package. Use yum or up2date command to install package
# yum install audit

or
# up2date install audit

Auto start auditd service on boot
# ntsysv

OR
# chkconfig auditd on

Now start service:
# /etc/init.d/auditd start

How do I set a watch on a file for auditing?
Let us say you would like to audit a /etc/passwd file. You need to type command as follows:
# auditctl -w /etc/passwd -p war -k password-file

Where,

-w /etc/passwd : Insert a watch for the file system object at given path i.e. watch file called /etc/passwd
-p war : Set permissions filter for a file system watch. It can be r for read, w for write, x for execute, a for append.
-k password-file : Set a filter key on a /etc/passwd file (watch). The password-file is a filterkey (string of text that can be up to 31 bytes long). It can uniquely identify the audit records produced by the watch. You need to use password-file string or phrase while searching audit logs.
In short you are monitoring (read as watching) a /etc/passwd file for anyone (including syscall) that may perform a write, append or read operation on a file.

Wait for some time or as a normal user run command as follows:
$ grep 'something' /etc/passwd
$ vi /etc/passwd

Following are more examples:

File System audit rules
Add a watch on “/etc/shadow” with the arbitrary filterkey “shadow-file” that generates records for “reads, writes, executes, and appends” on “shadow”
# auditctl -w /etc/shadow -k shadow-file -p rwxa

SYSCALL AUDIT RULE
The next rule suppresses auditing for mount syscall exits
# auditctl -a exit,never -S mount

FILE SYSTEM AUDIT RULE
Add a watch “tmp” with a NULL filterkey that generates records “executes” on “/tmp” (good for a webserver)
# auditctl -w /tmp -p e -k webserver-watch-tmp

SYSCALL AUDIT RULE USING PID
To see all syscalls made by a program called sshd (pid – 1005):
# auditctl -a entry,always -S all -F pid=1005

How do I find out who changed or accessed a file /etc/passwd?
Use ausearch command as follows:
# ausearch -f /etc/passwd

OR
# ausearch -f /etc/passwd | less

OR
# ausearch -f /etc/passwd -i | less

Where,

-f /etc/passwd : Only search for this file
-i : Interpret numeric entities into text. For example, uid is converted to account name.
Output:

----
type=PATH msg=audit(03/16/2007 14:52:59.985:55) : name=/etc/passwd flags=follow,open inode=23087346 dev=08:02 mode=file,644 ouid=root ogid=root rdev=00:00
type=CWD msg=audit(03/16/2007 14:52:59.985:55) :  cwd=/webroot/home/lighttpd
type=FS_INODE msg=audit(03/16/2007 14:52:59.985:55) : inode=23087346 inode_uid=root inode_gid=root inode_dev=08:02 inode_rdev=00:00
type=FS_WATCH msg=audit(03/16/2007 14:52:59.985:55) : watch_inode=23087346 watch=passwd filterkey=password-file perm=read,write,append perm_mask=read
type=SYSCALL msg=audit(03/16/2007 14:52:59.985:55) : arch=x86_64 syscall=open success=yes exit=3 a0=7fbffffcb4 a1=0 a2=2 a3=6171d0 items=1 pid=12551 auid=unknown(4294967295) uid=lighttpd gid=lighttpd euid=lighttpd suid=lighttpd fsuid=lighttpd egid=lighttpd sgid=lighttpd fsgid=lighttpd comm=grep exe=/bin/grep
Let us try to understand output

audit(03/16/2007 14:52:59.985:55) : Audit log time
uid=lighttpd gid=lighttpd : User ids in numerical format. By passing -i option to command you can convert most of numeric data to human readable format. In our example user is lighttpd used grep command to open a file
exe=”/bin/grep” : Command grep used to access /etc/passwd file
perm_mask=read : File was open for read operation
So from log files you can clearly see who read file using grep or made changes to a file using vi/vim text editor. Log provides tons of other information. You need to read man pages and documentation to understand raw log format.

Other useful examples
Search for events with date and time stamps. if the date is omitted, today is assumed. If the time is omitted, now is assumed. Use 24 hour clock time rather than AM or PM to specify time. An example date is 10/24/05. An example of time is 18:00:00.
# ausearch -ts today -k password-file
# ausearch -ts 3/12/07 -k password-file

Search for an event matching the given executable name using -x option. For example find out who has accessed /etc/passwd using rm command:
# ausearch -ts today -k password-file -x rm
# ausearch -ts 3/12/07 -k password-file -x rm

Search for an event with the given user name (UID). For example find out if user vivek (uid 506) try to open /etc/passwd:
# ausearch -ts today -k password-file -x rm -ui 506
# ausearch -k password-file -ui 506

Read man pages – auditd, ausearch, auditctl

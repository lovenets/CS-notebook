## Linux Kernel

### Definition 

- The kernel is a middle layer which virtualizes the common hardware resources of the computer to provide each process with its own virtual resource. This makes the process seem as it is the sole process running on the machine. The kernel is also responsible for preventing and mitigating conflicts between different processes.

### Core Subsystems of Linux Kernel

- The Process Scheduler: This kernel subsystem is responsible for fairly distributing the CPU time among all the processes running on the system simulteneously.
- The Memory Management Unit (MMU): This kernel sub-unit is responsible for proper distribution of the memory resources among the various processes running on the system. The MMU does more than just simply provide separate virtual address spaces for each of the processes.
- The Virtual File System (VFS): This subsystem is responsible for providing a unified interface to access stored data across different filesystems and physical storage media. 
  - Key directories in the file system:
    - */: Root directory (base of file system)

    - /bin: Executable programs

    - /boot: Linux kernel and boot loader

    - /dev: Special device files

    - /etc: System configuration files

    - /home: Home directories of all users

    - /lib: Library files for programs

    - /media: Mount points for CD-ROM and other media

    - /root: Home directory of the root user

    - */sbin: System administration commands

    - /srv: Data for services such as Web and FTP

    - */tmp: Temporary directory

    - /usr: Many of the important administration programs

    - /var: Various system files, such as logs
- The Networking Unit
- Inter-Process Communication Unit

## Common Linux Commands

### System Information

```bash
# Display Linux system information
uname -a

# Display kernel release information
uname -r

# Show which version of redhat installed
cat /etc/redhat-release

# Show how long the system has been running + load
uptime

# Show system host name
hostname

# Display the IP addresses of the host
hostname -I

# Show system reboot history
last reboot

# Show the current date and time
date

# Show this month's calendar
cal

# Display who is online
w

# Who you are logged in as
whoami
```

### Hardware Information

```bash
# Display messages in kernel ring buffer
dmesg

# Display CPU information
cat /proc/cpuinfo

# Display memory information
cat /proc/meminfo

# Display free and used memory ( -h for human readable, -m for MB, -g for GB.)
free -h

# Display PCI devices
lspci -tv

# Display USB devices
lsusb -tv

# Display DMI/SMBIOS (hardware info) from the BIOS
dmidecode

# Show info about disk sda
hdparm -i /dev/sda

# Perform a read speed test on disk sda
hdparm -tT /dev/sda

# Test for unreadable blocks on disk sda
badblocks -s /dev/sda
```

### Performance Monitoring And Statistics

```bash
# Display and manage the top processes
top

# Interactive process viewer (top alternative)
htop

# Display processor related statistics
mpstat 1

# Display virtual memory statistics
vmstat 1

# Display I/O statistics
iostat 1

# Display the last 100 syslog messages  (Use /var/log/syslog for Debian based systems.)
tail 100 /var/log/messages

# Capture and display all packets on interface eth0
tcpdump -i eth0

# Monitor all traffic on port 80 ( HTTP )
tcpdump -i eth0 'port 80'

# List all open files on the system
lsof

# List files opened by user
lsof -u user

# Display free and used memory ( -h for human readable, -m for MB, -g for GB.)
free -h

# Execute "df -h", showing periodic updates
watch df -h
```

### User Information Management

```bash
# Display the user and group ids of your current user.
id

# Display the last users who have logged onto the system.
last

# Show who is logged into the system.
who

# Show who is logged in and what they are doing.
w

# Create a group named "test".
groupadd test

# Create an account named john, with a comment of "John Smith" and create the user's home directory.
useradd -c "John Smith" -m john

# Delete the john account.
userdel john

# Add the john account to the sales group
usermod -aG sales john

# Change current user's password
passwd
```

### File And Directory

```bash
# List all files in a long listing (detailed) format
ls -al

# Display the present working directory
pwd

# Create a directory
mkdir directory

# Remove (delete) file
rm file

# Remove the directory and its contents recursively
rm -r directory

# Force removal of file without prompting for confirmation
rm -f file

# Forcefully remove directory recursively
rm -rf directory

# Copy file1 to file2
cp file1 file2

# Copy source_directory recursively to destination. If destination exists, copy source_directory into destination, otherwise create destination with the contents of source_directory.
cp -r source_directory destination

# Rename or move file1 to file2. If file2 is an existing directory, move file1 into directory file2
mv file1 file2

# Create symbolic link to linkname
ln -s /path/to/file linkname

# Create an empty file or update the access and modification times of file.
touch file

# View the contents of file
cat file

# Browse through a text file
less file

# Display the first 10 lines of file
head file

# Display the last 10 lines of file
tail file

# Display the last 10 lines of file and "follow" the file as it grows.
tail -f file

# Display file's attributes like type, owner, etc.
stat filename
```

### Process Management

```bash
# Display your currently running processes
ps

# Display all the currently running processes on the system.
ps -ef

# Display process information for processname
ps -ef | grep processname

# Display and manage the top processes
top

# Interactive process viewer (top alternative)
htop

# Kill process with process ID of pid
kill pid

# Kill all processes named processname
killall processname

# Start program in the background
programname &

# Display stopped or background jobs
bg

# Brings the most recent background job to foreground
fg

# Brings job n to the foreground
fg n
```

### File Permissions

![](https://www.linuxtrainingacademy.com/wp-content/uploads/2017/02/linux-permissions-chart.png)

| Permission  | Example            |
|-------------|--------------------|
| rwx rwx rwx | chmod 777 filename |
| rwx rwx r-x | chmod 775 filename |
| rwx r-x r-x | chmod 755 filename |
| rw- rw- r-- | chmod 664 filename |
| rw- r-- r-- | chmod 644 filename |

| Octal | Permission              | rwx | Binary |
|-------|-------------------------|-----|--------|
| 7     | read, write and execute | rwx | 111    |
| 6     | read and write          | rw- | 110    |
| 5     | read and execute        | r-x | 101    |
| 4     | read only               | r-- | 100    |
| 3     | write and execute       | -wx | 011    |
| 2     | write only              | -w- | 010    |
| 1     | execute only            | --x | 001    |
| 0     | none                    | --- | 000    |

### Networking

```bash
# Display all network interfaces and ip address
ifconfig -a

# Display eth0 address and details
ifconfig eth0

# Query or control network driver and hardware settings
ethtool eth0

# Send ICMP echo request to host
ping host

# Display whois information for domain
whois domain

# Display DNS information for domain
dig domain

# Reverse lookup of IP_ADDRESS
dig -x IP_ADDRESS

# Display DNS ip address for domain
host domain

# Display the network address of the host name.
hostname -i

# Display all local ip addresses
hostname -I

# Download http://domain.com/file
wget http://domain.com/file

# Display listening tcp and udp ports and corresponding programs
netstat -nutlp
```

### Archives

```bash
# Create tar named archive.tar containing directory.
tar cf archive.tar directory

# Extract the contents from archive.tar.
tar xf archive.tar

# Create a gzip compressed tar file name archive.tar.gz.
tar czf archive.tar.gz directory

# Extract a gzip compressed tar file.
tar xzf archive.tar.gz

# Create a tar file with bzip2 compression
tar cjf archive.tar.bz2 directory

# Extract a bzip2 compressed tar file.
tar xjf archive.tar.bz2
```

### Package Management

```bash
# Search for a package by keyword.
yum search keyword

# Install package.
yum install package

# Display description and summary information about package.
yum info package

# Install package from local file named package.rpm
rpm -i package.rpm

# Remove/uninstall package.
yum remove package

# Install software from source code.
tar zxvf sourcecode.tar.gz
cd sourcecode
./configure
make
make install
```

### Search

```bash
# Search for pattern in file
grep pattern file

# Search recursively for pattern in directory
grep -r pattern directory

# Find files and directories by name
locate name

# Find files in /home/john that start with "prefix".
find /home/john -name 'prefix*'

# Find files larger than 100MB in /home
find /home -size +100M
```

### SSH

```bash
# Connect to host as your local username.
ssh host

# Connect to host as user
ssh user@host

# Connect to host using port
ssh -p port user@host
```

### File Transfers

```bash
# Secure copy file.txt to the /tmp folder on server
scp file.txt server:/tmp

# Copy *.html files from server to the local /tmp folder.
scp server:/var/www/*.html /tmp

# Copy all files and directories recursively from server to the current system's /tmp folder.
scp -r server:/var/www /tmp

# Synchronize /home to /backups/home
rsync -a /home /backups/

# Synchronize files/directories between the local and remote system with compression enabled
rsync -avz /home server:/backups/
```

### Disk Usage

```bash
# Show free and used space on mounted filesystems
df -h

# Show free and used inodes on mounted filesystems
df -i

# Display disks partitions sizes and types
fdisk -l

# Display disk usage for all files and directories in human readable format
du -ah

# Display total disk usage off the current directory
du -sh
```

### Directory Navigation

```bash
# To go up one level of the directory tree.  (Change into the parent directory.)
cd ..

# Go to the $HOME directory
cd

# Change to the /etc directory
cd /etc
```
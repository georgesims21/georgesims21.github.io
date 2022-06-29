# procfs/sysfs

procfs and sysfs are both pseudo filesystems located in the root directory of the Unix FS heirarchy. They contain files which act as a *window* to the kernel and allows user-land processes to see and change important system information. 
Upon inspection (if you wrote the command `$ ls -al /proc` on a Unix system) you would notice that all files are 0 in size. This is due to the *window* aspect, one a user-land process reads/opens one of these files, it is at that moment the kernel writes that information to the file, hence the analogy. Imagine you are out shopping, where the shops changed their items every second, and only when you look through the window will the shop owner place their items in view for you to see, saving the shop owner a lot of energy if you only decide to look once every minute.
  There are files containing singular process information as well as system wide information, and procfs uses symlinks to allow processes to automatically see stats releated to the caller, without the caller making extra effort.

sysfs contains files which interact with the kernel and allows user-land processes to modify kernel parameters without entering the kernel themselves. Processes can write to these files and the kernel will update the parameters accordingly within the kernel.
<full sysfs info here once completed research>

The aim of my bachelor project is to create a filesystem using FUSE which mirrors procfs and sysfs for a distributed setting. To begin planning, it is important to realise which filesystem operations I will actually need to implement, and which ones I should leave out.
After checking the [linux/fs/proc/generic file](https://github.com/torvalds/linux/blob/6f0d349d922ba44e4348a17a78ea51b7135965b1/fs/proc/generic.c) (which contains the main filesystem operations for procfs on the master branch) I found that these were the ones given to the VFS by procfs:

* getattrb
* readdir
* llseek
* read
* lookup
* setattrb
* create
* symlink
* mkdir
* open
* release
* remove

Apart from llseek and the remove operation, all of these methods are offered by the FUSE api to be used within a new FS. Due to my bachelor project *mirroring* procfs, operations which modify files/directories in any way should not be implemented, thus here are the final operations which I must implement:

* getattrb
* readdir
* read
* lookup
* open
    test


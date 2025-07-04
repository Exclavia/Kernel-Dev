# 8. The VFS and the initrd
In this chapter we're going to be starting work on our virtual filesystem (VFS). As a baptism of fire, we will also be implementing an initial ramdisk so you can load configuration files or executables to your kernel.

## 8.1. The virtual filesystem
A VFS is intended to abstract away details of the filesystem and location that files are stored, and to give access to them in a uniform manner. They are usually implemented as a graph of nodes; 
<img src="https://raw.githubusercontent.com/Exclavia/Kernel-Dev/refs/heads/main/assets/vfs.png" >

Each node representing either a file, directory, symbolic link, device, socket or pipe. Each node should know what filesystem it belongs to and have enough information such that the relavent open/close/etc functions in its driver can be found and executed. A common way to accomplish this is to have the node store function pointers which can be called by the kernel. We'll need a few function pointers:
- Open - Called when a node is opened as a file descriptor.
- Close - Called when the node is closed.
- Read - I should hope this was self explanatory!
- Write - Same as above :-)
- Readdir - If the current node is a directory, we need a way of enumerating it's contents. Readdir should return the n'th child node of a directory or NULL otherwise. It returns a 'struct dirent', which is compatible with the UNIX readdir function.
- Finddir - We also need a way of finding a child node, given a name in string form. This will be used when following absolute pathnames.

So far then our node structure looks something like:
```c
typedef struct fs_node
{
  char name[128];     // The filename.
  u32int flags;       // Includes the node type (Directory, file etc).
  read_type_t read;   // These typedefs are just function pointers. We'll define them later!
  write_type_t write;
  open_type_t open;
  close_type_t close;
  readdir_type_t readdir; // Returns the n'th child of a directory.
  finddir_type_t finddir; // Try to find a child in a directory by name.
} fs_node_t;
```
Obviously we need to store the filename, and flags contains the type of the node (directory, symlink etc), but we are still missing things. We need to know what permissions the file has, which user/group it belongs to, and possibly also its length.
```c
typedef struct fs_node
{
   char name[128];     // The filename.
   u32int mask;        // The permissions mask.
   u32int uid;         // The owning user.
   u32int gid;         // The owning group.
   u32int flags;       // Includes the node type.
   u32int length;      // Size of the file, in bytes.
   read_type_t read;
   write_type_t write;
   open_type_t open;
   close_type_t close;
   readdir_type_t readdir;
   finddir_type_t finddir;
} fs_node_t;
```
Again though, we are still missing things! We need a way for the filesystem driver to track which node is which. This is commonly known as an inode. It is just a number assigned by the driver which uniquely represents this file. Not only that, but we may have multiple instances of the same filesystem type, so we must also have a variable that the driver can set to track which filesystem instance it belongs to.

Lastly we also need to account for symbolic links (shortcuts in Windows-speak). These are merely pointers or placeholders for other files, and so need a pointer member variable.
```c
typedef struct fs_node
{
   char name[128];     // The filename.
   u32int mask;        // The permissions mask.
   u32int uid;         // The owning user.
   u32int gid;         // The owning group.
   u32int flags;       // Includes the node type. See #defines above.
   u32int inode;       // This is device-specific - provides a way for a filesystem to identify files.
   u32int length;      // Size of the file, in bytes.
   u32int impl;        // An implementation-defined number.
   read_type_t read;
   write_type_t write;
   open_type_t open;
   close_type_t close;
   readdir_type_t readdir;
   finddir_type_t finddir;
   struct fs_node *ptr; // Used by mountpoints and symlinks.
} fs_node_t;
```
### 8.1.1. Mountpoints

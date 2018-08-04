populatefs 1.1
==========

A nearly drop-in replacement for `genext2fs` from [tytso/e2fsprogs](https://github.com/tytso/e2fsprogs) that is compatible with ext4.  

version 1.1+ has added support for [offset images](https://github.com/oskarirauta/populatefs/issues/4)

Image generation
================
To produce a 42MiB filesytem image:

```
label=OpenWrt
inodes=8096
fsopts=has_journal,ext_attr,resize_inode,dir_index,filetype,extent,flex_bg,sparse_super,large_file,uninit_bg,dir_nlink,extra_isize

dd if=/dev/zero of=root.ext4 seek=42720 count=0 bs=1k
mkfs.ext4 -F -E root_owner=0:0 -L $label] -N $inodes -O $fsopts -m 0 root.ext4
```

then use populatefs to populate the empty filesystem:
`populatefs -U -d [directory containing files to go into root] -D devtable.txt root.ext4`

Usage
=====
```
Usage:./populateds [options] (image | diskimage?offset=<starting-byte-of-ext4-partition>)"
Manipulate ext4 images from directories/files

 -d <directory>   Add the given directory and contents at a particular path to root
 -D <file>        Add device nodes and directories from filespec
 -U               Squash owners making all files be owner by root:root
 -P               Squash permissions on all files
 -q               Same as "-U -P"
 -S <number>      Shift UIDs & GIDs by decrementing S-many from source value
 -h               Display this usage guide
 -V               Show version
 -v               Be verbose
 -w               Be even more verbose
```

Advanced use
============
You can add multiple directories of filespecs with multiple -d or -D options.  
These are processed in the order you give 'em to command-line.  

If these sources contain pathnames that will overlap:
*  previously added files will be overwritten by new files.  
*  previously added directories will include new source's files.  
*  error will be printed out if adding a directory and that pathname is already a file.  

Rules are simple: do not overwrite directory with file, or file with directory.  

Offset-images ("disk images") can be handled using some [under-documented](https://github.com/tytso/e2fsprogs/blob/master/doc/RelNotes/v1.36.txt#L116) [syntax](https://github.com/tytso/e2fsprogs/commit/2e8ca9a26b0bd7dae546a3f9a98df67b043fe3be) to provide the starting byte of the EXT superblock.  

The same offset can be used to perform a loop-mount to verify the image contents.  

Compilation
===========
No autohell, just a Makefile with overridable variables.

`CC=gcc make`
or
`CC=gcc DESTDIR=/tmp make install`

Study the values from Makefile.
If your system did not ship or does not have getopt.. Try:
`EXTRA_CFLAGS="" make`

Requirements
============
libext2fs and libmath must be available for compilation.  
* Debian: `sudo apt-get install libext2fs-dev libc6-dev`

For runtime, kernel must have support for debugfs.  

Credits
=======
This is a fork from [oskarirauta/populatefs 0.9](https://github.com/oskarirauta/populatefs)
& [lipnitsk/populatefs 1.0](https://github.com/lipnitsk/populatefs)  


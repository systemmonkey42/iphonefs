# iPhoneBackupFS

A golang project to mount an __unencrypted__ iPhone backup.

Tested on linux (ubuntu-20.06 and later) and Windows 10 (21H2)

# Building and Installation

Use `git clone` to obtain the default branch (develop) and install __go 1.19+__

## Dependencies

The following packages (when building on ubuntu) are needed:
- fuse3
- libfuse3-dev

The following are required when building on windows:

- [WinFSP](https://winfsp.dev/rel/) (Make sure to select fuse SDK/Development files when installing)
- [MinGW-w64 Project](https://www.mingw-w64.org/)
  - I recommend installing the latest from GitHub release page. (https://github.com/niXman/mingw-builds-binaries/releases)

After installation, you may need to set the following in your environment for `go build` to be able to locate WinFSP


```
set CFLAGS=C:\Program Files (x86)\WinFsp\inc\fuse
```

## Building

Build with the following command:


```
go build -tags &lt;mode&gt; .
```


Where __mode__ is one of
- `bazil` for the `bazil/fuse` library (fast, linux only) or
- `winfsp` for the windows compatible implementation.


**Note**: New features will only be added to the winfsp implementation, which will eventually become the default when building

If you have issues building (such as when building in a container), or simply want a smaller executable, try the following:

```
go build -tags osusergo,netgo -o iphonebackupfs -ldflags "-w -s" .
```

Move the resulting binary to somewhere accessible

```
sudo mv iphonebackupfs /usr/local/bin
```

# Usage

```
iphonebackupfs [-A] [-L] [-d <domain>] <backup folder> <mount point>
```

The default mode will present the camera roll at the root of the mount point.  The is the quickest and simplest way to connect and extract images and videos.

To list the available domains, use the "-L" parameters.

To specify a domain, use the `-d <domain>` option. For example, to mount the SMS application and get access to received attachments folder, use the domain `MediaDomain` and browse to the `Library/SMS/Attachments` folder.

To mount the entire backup, use `-A`.  The will cause all domain names to become part of the filesystem.


```
iphonebackupfs /path/to/directory/containing/backup  /mnt/path
```


Note that the backup directory should contain a file called "Metadata.db".

By default, pressing <kbd>Ctrl-C</kbd> will attempt to dismount the filesystem.  Under linux, you can manually unmount the filesystem to terminate the application with:


```
umount /mnt/path

```

## Environment Variables

The following environment variables are used when starting the application

Name|Description
---|---
ROOT|Root of the backup folder (directory containing manifest.db
MOUNT|Directory to use as mountpoint

Note: To use `$ROOT` but specify a mount point on the command line, specify the empty string `''` as the backup folder.
For example:
```
iphonebackupfs '' /mnt/data
```


# Issues

- All files are readonly
- The iphone metadata is read once prior to making the entire filesystem available, and is never referenced again.
- Metadata inside the backup is ignored.  File timestamps default to current time.
- All backup files are classified into "domains".  By default, only the "CameraRollDomain" is mounted.
- iPhone applications make use of sqlite databases, however opening a sqlite database on a read-only filesystem requires the alternate "url" format with the __immutable__ option set (eg: `file://path/to/sqllite.db?immutable=1`)


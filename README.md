# Minizip 2.0.0

This library is a refactoring of the minizip contribution found in the zlib distribution that is supported on Windows, macOS, and Linux. It is based on the original work of [Gilles Vollant](http://www.winimage.com/zLibDll/minizip.html) that has been contributed to by many people over the years.

Dev: ![Dev Branch Status](https://travis-ci.org/nmoinvaz/minizip.svg?branch=dev)
Master: ![Master Branch Status](https://travis-ci.org/nmoinvaz/minizip.svg?branch=master)

For the older release of the library checkout the [1.2](https://github.com/nmoinvaz/minizip/tree/1.2) branch.

## Build

To generate the project files for your platform and IDE download and run cmake in the project directory.

```
cmake .
cmake --build .
```

## Contents

| File(s) | Description |
|:- |:-|
| miniunz.c | Sample unzip application |
| minizip.c | Sample zip application |
| mz_compat.\* | Minizip 1.0 compatibility layer |
| mz_error.h | Error codes for all the functions |
| mz_os\* | OS specific helper functions |
| mz_strm.\* | Stream interface |
| mz_strm_aes.\* | WinZIP AES stream |
| mz_strm_buf.\* | Buffered stream |
| mz_strm_bzip.\* | BZIP2 stream using libbzip2 |
| mz_strm_crypt.\* | PKWARE traditional encryption stream |
| mz_strm_lzma.\* | LZMA stream using liblzma |
| mz_strm_mem.\* | Memory stream |
| mz_strm_split.\* | Disk splitting stream |
| mz_strm_posix.\* | File stream using Posix functions |
| mz_strm_win32.\* | File stream using Win32 API functions |
| mz_strm_zlib.\* | Deflate stream using zlib |
| mz_unzip.\* | Unzip functionality |
| mz_zip.\* | Zip functionality |

## Features

### Streams

This library has been refactored around streams.

#### Memory Streaming

To unzip from a zip file in memory create a memory stream and pass it to the unzip open functions.
```
uint8_t *zip_buffer = NULL;
int32_t zip_buffer_size = 0;
void *mem_stream = NULL;

// fill zip_buffer with zip contents
mz_stream_mem_create(&mem_stream);
mz_stream_mem_set_buffer(mem_stream, zip_buffer, zip_buffer_size);
mz_stream_open(mem_stream, NULL, MZ_STREAM_MODE_READ);

void *unz_handle = mz_unzip_open(mem_stream);
// do unzip operations

mz_stream_mem_delete(&mem_stream);
```

To create a zip file in memory first create a growable memory stream and pass it to the zip open functions.

```
void *mem_stream = NULL;

mz_stream_mem_create(&mem_stream);
mz_stream_mem_set_grow(mem_stream, 1);
mz_stream_mem_set_grow_size(mem_stream, (128 * 1024));
mz_stream_open(mem_stream, NULL, MZ_STREAM_MODE_CREATE);

void *zip_handle = mz_zip_open(0, 0, mem_stream);
// do unzip operations

mz_stream_mem_delete(&mem_stream);
```
#### Buffered Streaming

By default the library will read bytes typically one at a time. The buffered stream allows for buffered read and write operations to improve I/O performance.

```
void *stream = NULL;
void *buf_stream = NULL;

mz_stream_os_create(&stream)
// do open os stream

mz_stream_buffered_create(&buf_stream);
mz_stream_buffered_open(buf_stream, NULL, MZ_STREAM_MODE_READ);
mz_stream_buffered_set_base(buf_stream, stream);

void *unz_handle = mz_unzip_open(buf_stream);
```

#### Disk Splitting Stream

To create an archive with multiple disks use the disk splitting stream and for zipping supply a disk size value in bytes.

```
void *stream = NULL;
void *split_stream = NULL;

mz_stream_os_create(&stream);

mz_stream_split_create(&split_stream);
mz_stream_split_set_prop_int64(split_stream, MZ_STREAM_PROP_DISK_SIZE, 64 * 1024);

mz_stream_set_base(split_stream, stream);

mz_stream_open(split_stream, path..

handle = mz_unzip/zip_open(split_stream);
```

The central directory is the only data stored in the .zip and doesn't follow disk size restrictions.

When unzipping it will automatically determine when in needs to cross disk boundaries.

### Compression Methods

#### BZIP2

+ Requires #define HAVE_BZIP2
+ Requires [BZIP2](http://www.bzip.org/) library

#### LZMA

+ Requires #define HAVE_LZMA
+ Requires [liblzma](https://tukaani.org/xz/) library

### Encryption

#### [WinZIP AES Encryption](http://www.winzip.com/aes_info.htm)

+ Requires #define HAVE_AES
+ Requires [Brian Gladman's](https://github.com/BrianGladman/aes) AES library

When zipping with a password it will always use AES 256-bit encryption.
When unzipping it will use AES decryption only if necessary. Does not support central directory or local file header encryption since it is not supported outside of PKZIP. For a more secure method it is best to just encrypt the zip post-process.

### Windows RT

+ Requires #define IOWIN32_USING_WINRT_API

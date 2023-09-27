- [Current status](#current-status)
- [Brief Summary](#brief-summary)
- [Stakeholders](#stakeholders)
- [Technical Design - Version 1](#technical-design---version-1)
  - [Changes From CryPak](#changes-from-crypak)
  - [New Features](#new-features)
    - [Format Features](#format-features)
    - [Implementation Features](#implementation-features)
  - [What's Not Supported?](#whats-not-supported)
  - [Technical Explanation](#technical-explanation)
    - [Archive Format Decisions](#archive-format-decisions)
      - [Storing paths of files within the archive WITHOUT of file paths hashes](#storing-paths-of-files-within-the-archive-without-of-file-paths-hashes)
      - [Fixed Header up front / Variable TOC at end](#fixed-header-up-front--variable-toc-at-end)
      - [File Globbing not directly supported in Archive System](#file-globbing-not-directly-supported-in-archive-system)
    - [How do individual file operations of adding, removing and modifying a file within the Archive work?](#how-do-individual-file-operations-of-adding-removing-and-modifying-a-file-within-the-archive-work)
      - [Adding a File](#adding-a-file)
      - [Removing a File](#removing-a-file)
      - [Updating a File](#updating-a-file)
    - [How to create a minimal archive with no gaps?](#how-to-create-a-minimal-archive-with-no-gaps)
      - [Copying an Archive](#copying-an-archive)
    - [How to extract a file from the archive(For Tools)?](#how-to-extract-a-file-from-the-archivefor-tools)
      - [Extracting a File](#extracting-a-file)
    - [How are file path lookup's resolved when multiple archives are mounted?](#how-are-file-path-lookups-resolved-when-multiple-archives-are-mounted)
    - [Technical API](#technical-api)
      - [Archive Format Data Layout](#archive-format-data-layout)
      - [Decompression Interface API](#decompression-interface-api)
      - [Compression Interface API](#compression-interface-api)
      - [Archive In-Memory API](#archive-in-memory-api)
      - [Archive Reader API](#archive-reader-api)
      - [Archive Writer API](#archive-writer-api)
      - [Archive Streamer Integration API](#archive-streamer-integration-api)
      - [Streamer API Updates](#streamer-api-updates)
  - [Scope](#scope)
    - [AzCore Streaming System](#azcore-streaming-system)
    - [Gem System](#gem-system)
    - [Asset Bundler](#asset-bundler)
  - [Testing plan](#testing-plan)
  - [Execution Plan (And Dependencies)](#execution-plan-and-dependencies)
    - [Phase 1](#phase-1)
    - [Phase 2](#phase-2)
    - [Post-Launch](#post-launch)
    - [Other Dependencies](#other-dependencies)
      - [Asset Bundler Preset Feature](#asset-bundler-preset-feature)
    - [Migration Plans (Data, and workflows)](#migration-plans-data-and-workflows)
  - [Risks](#risks)
  - [Alternatives](#alternatives)
    - [Update CryPak to support the new Archiving Features](#update-crypak-to-support-the-new-archiving-features)
        - [Benefits](#benefits)
        - [Downsides](#downsides)
    - [Use ZIP64 via libzip for the Archive format](#use-zip64-via-libzip-for-the-archive-format)
      - [Benefits](#benefits-1)
      - [Downsides](#downsides-1)
    - [Use the XAR format via the xar library](#use-the-xar-format-via-the-xar-library)
      - [Benefits](#benefits-2)
      - [Downsides](#downsides-2)
  - [FAQ](#faq)
    - [Why not just update CryPak(AZ::IO::Archive) to support these features?](#why-not-just-update-crypakazioarchive-to-support-these-features)
    - [What about using the XAR format?](#what-about-using-the-xar-format)
    - [Do we have a new name for the new archive solution besides "O3AR"?](#do-we-have-a-new-name-for-the-new-archive-solution-besides-o3ar)
    - [Would the new Archive System have any effect on the deprecating AZ FileIO API?](#would-the-new-archive-system-have-any-effect-on-the-deprecating-az-fileio-api)

Current status
==================

Pending Review

Brief Summary
=================

A new Archiving System will be introduced to O3DE which replaces the AZ::IO::Archive(formally CryPak) Archiving System.  
For the rest of this document AZ::IO::Archive will be referred to as CryPak, despite it being [renamed and moved to AzFramework](https://github.com/o3de/o3de/tree/development/Code/Framework/AzFramework/AzFramework/Archive)

This document is based on the [Streaming Stack Archive](./rfc-core-2023-04-20-streaming-archive.md) which details the structure of the archiving format.  
Feature Parity with CryPaks not the goal of the new Archiving System nor should it be as the system will focus primarily on the archiving aspect.  
CryPak design has several issues and complexities which makes it not suitable for streaming content off of modern SSDs and NVMe devices at high speed(> 2000MiB/s)  
  
The new archiving format is the designed with the ability to take advantage of PCIe 4.0 speeds which can support up to [33GiB](https://en.wikipedia.org/wiki/PCI_Express#History_and_revisions) throughput with devices already in the [market](https://www.xpg.com/us/xpg/685?tab=specification) that supports read speeds of over 7000Mib/s.  
Furthermore customers who are looking to use their own compression solution will be able to extend O3DE with support for additional compression algorithms or choose to forgo compression altogether

Stakeholders
================

| Team Name | Why? |
| --- | --- |
| sig-platform | Any hardware optimizations of reading an archive file would need involvement from the platform team, particularly for consoles and their NVMe access patterns  <br/>Implementation of hardware decompressions would most likely be done by the platform group |
| sig-core (streaming) | The Archive System will be added to AZ::IO:Streamer stack to allow archives to read at runtime. Only reading will be supported by the Streaming system |
| sig-content (asset pipeline) | Tools such as the AssetBundler will need to be updated to read the new archive format as well as being able to create archive files itself |

Technical Design - Version 1  
===================================

Changes From CryPak
------------------------

The Archiving System API will not try to model a filesystem like API that current CryPak does via the [ArchiveFileIO](https://github.com/o3de/o3de/blob/5f5dddf46a8a2b6609c2088e70793e89f5ff48dd/Code/Framework/AzFramework/AzFramework/Archive/ArchiveFileIO.h#L22-L28) class  
It will not have the following as part of it's API

*   The ability to Create/Remove Directories within the Archive
*   Querying FilesystemAttributes (type\[directory, file, symlink\], modtime, permissions,etc...) within the archive
*   Search for files via globbing directly as part of the archive API  
    This can be done at a higher level with another entry in the Streamer Stack, that builds a directory tree out of the mounted archive files  
    
*   Resolving FileIOAliases  
    i.e @product@/level.archive/asset.txt, → D:/o3de/AutomatedTesting/level.archive/asset.txt

The CryPak archive API can create and query ZIP Format, with support for 3 hard-coded Compression Codecs.  
The use of the Compression Algorithms of Zlib, LZ4, ZSTD, will NOT be implemented as part of the new Archive API system itself,  
but instead layered as an entry in the Streamer that can perform decompression after streamer has read the data from an Archive file

Searching for Files within the multiple mounted Archive will be managed by the Archive Streamer stack node, that will provide a mechanism for prioriting file lookups within the archive itself.  
The search priority between loose files and archive files, will be completely kept out of the Archive System, and will take advantage of the how nodes are order in Streamer stack.  
As the Streamer stack is configurable through the Settings Registry, this is user controllable.  

New Features
-----------------

The following features would be added for the archive format

### Format Features

1.  Support for Archive files over 4GiB  
    The limit for single archive file would be up to 170TiB, with support for up to 33 million files.  
    CryPak currently only supports the standard ZIP Format and not ZIP64, despite the [ZipFileFormat.h](https://github.com/o3de/o3de/blob/5f5dddf46a8a2b6609c2088e70793e89f5ff48dd/Code/Framework/AzFramework/AzFramework/Archive/ZipFileFormat.h#L129-L141) header implying such support.  
    Only 32-bit compressed and uncompressed sizes are [read](https://github.com/o3de/o3de/blob/5f5dddf46a8a2b6609c2088e70793e89f5ff48dd/Code/Framework/AzFramework/AzFramework/Archive/Archive.cpp#L107) from a compressed entry with no support for 64-bit values.  
2.  Archive header stored at the beginning of the Archive  
    This allows mounting the archive to only read the uncompressed section of the archive header(which is 80 bytes currently padded up to 512-bytes) for future entries.  
    This can then be used to located the Table of Contents at the end of the Archive in a single seek.
3.  The Archive Table of Contents (TOC) is stored at the end of the Archive file  
    There is a TOC at the end of the Archive File that is variable length.  
    The TOC is optionally compressed which will be indicated within the Archive header  
    The Archive format supports up to 2^25 or ~33 million files(This is a soft cap in the format, technically up to 2^32 can be stored, but it will be restricted to 2^25 for now)  
    So for an archive with the maximum amount of ~33 million files, around 640MiB(671088688 bytes) of uncompressed data will need to be loaded into memory  
    as opposed to reading upwards of 128TiB to locate a file within the archive.
4.  Support for multiple compression algorithms in a single archive  
    Up to 7 different compression algorithms will be supported in a single archive.
5.  FilePaths are stored within the TOC  
    The new archive format will store the paths to files within the TOC of the Archive File. It will support paths up to(2^16-1) 65535 bytes

### Implementation Features

1.  Block-level Reads/Decompression  
    Files will be compressed at the block level of entire file level.  
    Every 2MiB block of a file will be compressed individually.  
    Compressed data will be aligned on 512 byte boundaries.  
    This has the benefit to allow both reads and decompression to occur in parallel.  
    It also compliments well with Streamer read request API to read blocks of files in parallel as well.  
    Furthermore memory needs are able to be reduced by allowing streaming in of blocks instead of entire files  
    The reason why blocks of 2MiB are compressed indvidually and compressed data being aligned on 512 byte boundaries are explained more in detail within the [Streaming Stack Archive - Technical design](./rfc-core-2023-04-20-streaming-archive.md#technical-design) section.
2.  Use of the Job System to parallelize read/decompression request via Streamer  
    With Block level compression support, reads and decompression of blocks can run in parallel using the Job System
3.  Support for "overlay" files with the same relative path from multiple archives  
    Archive files that are mounted later will have there File Paths searched first when locating a file.  
    This allows later archives to overlay files from earlier archives(useful for DLC).

What's Not Supported?
--------------------------

*   The ability to mount archives within archives.  
    There is not really a good use case for this workflow and is very complicated to implement(It would require keeping the inner archive entirely in memory)  
    An alternative approach from nesting files is to take files in the inner archive and add them to the outer archive as flat structure.  
    
*   The ability to store files that differ only by case.  
    Since O3DE supports both case-preserving filesystems(exFAT, NTFS, APFS, ...) and case-sensitive filesystems(Ext4, Btrfs, ...) an archive created on one filesystem needs to work on another.  
    In order to allow that behavior, any relative paths that are added to the archive are always-lowercase.  
    

Technical Explanation
--------------------------

In order to use the new Archive format with the Streaming Stack several systems will need to be in place.

1.  The new Archive File format will need a Writer and Reader for it to exist.
2.  A streamer stack entry will be implemented that that creates Archive Reader instances to read files and metadata from the archive
3.  A Mount and Unmount Command will be added to the IStreamer API to support interfacing with the new streamer stack entry for reading archives
4.  The Streamer File Request Command Data will need to be updated to support a payload for the Mount and Unmount command
5.  A new decompression Interface will need to be added to allow the registration of compression algorithms decompression support from gems.
6.  Also a new compression Interface will need to be added to allow registration of compression algorithms write support for Tooling applications.

### Archive Format Decisions

The new Archive format makes several decisions with in regards to what metadata is stored within the Archive File, as well as where the Table of Content is located.

#### Storing paths of files within the archive WITHOUT of file paths hashes

The reasons that file paths are stored within an archive are plentiful.  
Storing the file paths within the archive helps debugability of the archive if the something goes wrong with the reading the file.  
As this is custom format being implemented within the O3DE engine, there aren't existing tools available to view an this format, so having the file paths of files stored within the Archive was preferred to help debug when bugs occur.  
While storing file paths take a lot more space than storing the hashes file pathes, it opens up the possibility to transform the file path into a structure that users can query.  
For example a directory structure can be made out of the file paths stored within all mounted archive files to allow support for file globbing.  
This also doesn't preclude the in-memory archive format from hashing the file paths when the archive file is mounted to save memory if need be in a later revision of the format.  
Moreover not storing file paths hashes within the archive avoids the issue of dealing with hash collisions during creation of the archive file itself.  
Finally the implementation for supporting File path hashes becomes a lot more complex(The need to deal with hash collisions, within the archive as well across multiple archives, determining which files in a base archive is replaced by it's overlay if the hashes are different, etc..)

There are some benefits to storing a pre-calculated table of file path hashes in that mounting an Archive could be faster if a file path hash table is preferred to be used at runtime.  
  
Ultimately it was decided that storing the file paths instead of hashes within the for the Archive format V1.

#### Fixed Header up front / Variable TOC at end

The Archive format goes with a header at the front and Table of Contents at the end as hybrid approach for retrieving information about files within the Archive.  
Using a fixed size header up front, allows the archive format to store information about the the archive file, such as it's version, the compression algorithms being used and most importantly the offset and size of the TOC.  
Individual file data is stored right after the fixed size header. As the header is fixed, modifying it doesn't require the file data within the archive to be moved.  
The TOC being at the end of the file allows individual files within the Archive to be added/removed or modified, without the need move ALL file data within the archive.  
An operation such as adding a new file would only need to update the fixed size header, add the data for the file in an existing gap with the Archive or at the end and finally write out the new Table of Contents afterwards.  
Since the Archive File itself can be on the order of several hundreds MiB to tens GiB, writing out a single file plus a TOC on the order of kilobytes/megabytes is a lot cheaper.

#### File Globbing not directly supported in Archive System

The ability to search for files via a wildcard(globbing) can be useful systems looking to quickly determine the location of files that it wishes to open without knowing.  
The Archive System would not directly support globbing as part of it's API.  
Instead it the Archive Reader Streamer Stack Entry would provide access to the table of file paths to mounted archive index + path index within the mounted archive.  
That data can be used by a higher level system to build up a directory structure that can support globbing of files.  
A data structure such as a [Radix Tree](https://en.wikipedia.org/wiki/Radix_tree) can be generated from the table of mounted archive file paths to create a directory structure for searching for patterns.  
Each edge could encode one or more path segments.  
Such a system could be built as another node within the Streamer stack if need be.

### How do individual file operations of adding, removing and modifying a file within the Archive work?

Because the variable length Table of Contents for the archive is at the end, operations such as adding/removing and updating the data of a file can be localized to affect only that section of the Archive File data.

#### Adding a File

The TOC supports linking unused/free blocks together to allow them to be re-used.

1.  Load the header and TOC into memory
2.  Use the headers first deleted block offset entry to build an in-memory sorted set of deleted blocks
    1.  The first 8 bytes of each deleted block contains the next deleted block offset or `0xffff'ffff'ffff'ffff`
3.  Depending on if the file being added can fit within the deleted blocks,
    1.  The new file data would be added to those deleted block
        1.  Locate the last deleted block before the location where the file data will be written to block
        2.  Write the file data to the deleted blocks
        3.  Locate the first deleted block offset after the location where the file data was written using the deleted block set
        4.  Write to the first 8 bytes of the deleted block before the newly added file payload, the offset of the first deleted block afterwards
        5.  Remove the blocks the added file used from the deleted block set
    2.  Otherwise the file data would be added to the offset where the table of contents was read
4.  Write out the content to the new location(optionally sending it through compression algorithm)
5.  Update the TOC with the new file metadata (and the TOC free blocks section if it has one)
6.  Write the TOC at the end of the archive.

#### Removing a File

For speedy file removal operations, the actual file data within the Archive wouldn't be touched, only the TOC would be updated to remove that file metadata.  
After the file metadata is removed, it's block entries are updated to be part of the free block list in the TOC

1.  Load the header and TOC into memory
2.  Use the headers first deleted block offset entry to build an in-memory sorted set of deleted blocks
    1.  The first 8 bytes of each deleted block contains the next deleted block offset or `0xffff'ffff'ffff'ffff`
3.  Locate the start of the file block data using the file metadata
    1.  Track each file deleted block offset in a local set
    2.  For every block except the last of the file data write out the next block offset in the first 8 bytes of the block
    3.  For the last block, write the offset of the first deleted block in the deleted block set that is at a greater offset(upper\_bound)
    4.  Insert the local deleted block offsets into the global deleted block set
4.  Update the TOC to remove that file metadata and path index.
5.  Write out the TOC at the end of the archive

#### Updating a File

For updating there two different approaches that need to be done depending on if content the file ultimately grew in size or shrunk.  
If the file grew it would pretty much have to following the [Adding a file](#adding-a-file) section above followed.

1.  Load the header and TOC into memory
2.  Use the headers first deleted block offset entry to build an in-memory list of deleted blocks
    1.  The first 8 bytes of each deleted block contains the next deleted block offset or `0xffff'ffff'ffff'ffff`
3.  Depending on if the file being added can fit within the deleted blocks,  
    1.  The updated file data would be added to those free blocks
        1.  Locate the last deleted block before the location where the file data will be written to block
        2.  Write the file data to the deleted blocks
        3.  Locate the first deleted block offset after the location where the file data was written using the deleted block set
        4.  Write to the first 8 bytes of the deleted block before the newly added file payload, the offset of the first deleted block afterwards
        5.  Remove the blocks the added file used from the deleted block set
    2.  Otherwise the file data would be added to the location where the table of contents starts
4.  Write out the updated content to the new location
5.  Update the file metadata for the modified file within the TOC
    1.  The previous blocks the file were using is updated in the TOC contents to be part of the free blocks list
6.  Write out the TOC at the end of the archive

If the file grew smaller , then some the file data can be overridden in the existing blocks and

1.  Load the header and TOC into memory
2.  Use the headers first deleted block offset entry to build an in-memory list of deleted blocks
    1.  The first 8 bytes of each deleted block contains the next deleted block offset or `0xffff'ffff'ffff'ffff`
3.  Locate the file's current block start offset and write out the updated content that to those blocks.
4.  For any blocks that are now unused for the current file
    1.  Track each file as unused block offset in a local set
    2.  For each of these blocks except the last write out the next block offset in the first 8 bytes of the block
    3.  For the last block in the local set, write the offset of first deleted block in the deleted block set that is at a greater offset than the current block
    4.  Insert the local deleted block set offset into the global deleted block set
5.  Update the file metadata with the new size info in the TOC.
6.  Write out the TOC at the end of the archive.

### How to create a minimal archive with no gaps?

After creation of a new archive it has the optimal minimum size with no free blocks within it.  
However individual operations of add, remove and updated performed after creation of the archive can leave unused blocks of data.  
The mechanism for creating a minimum sized archive from such a gap-filled archive in Version 1 would involve, making a new archive out of the existing archive using it's Header and TOC  
In this scenario the optimizations will be performed when creating the new archive such as skipping decompressing and re-compressing the archive content in order to copy the content over.

#### Copying an Archive

For making a new archive out of an existing one, an archive copy operation can be supported.

1.  Create the new archive in-memory with an empty TOC
2.  Read the header and load the entire TOC into memory for the existing archive
3.  Copy over the existing archive header to the new archive
    1.  Clear out the deleted file index and block index entries from the header
4.  Iterate over each file's path index and metadata entries in lock step
5.  Depending on if the path index entry deleted bit is set
    1.  If the deleted bit is not set copy both the file path index and metadata entries to the new archive TOC
        1.  The file path index is used to locate the file path bytes to copy over to the new TOC file path table
        2.  The file metadata is used to locate the blocks offsets entries which is copied over to the new TOC block offset table
        3.  The blocks for the file are copied over to the end of the new archive
    2.  If the deleted bit is set, then the file is skipped
6.  Write TOC to the end of the new archive
7.  Write new archive to disk.

### How to extract a file from the archive(For Tools)?

There will be a need for users to be extract files from the archive format during development.  
This may be needed for auditing or debugging content within the Archive.

#### Extracting a File

The extraction operations is would be be a similar path for the reading of archive data at runtime.  
There is an additional step of writing that content to a file path supplied by the user or defaulting to the extracted filename path segment

1.  Read the header and load the entire TOC into memory for the existing archive
2.  Iterate over each file's path index and metadata entries in lock step
3.  For each file's path index entry that isn't deleted read the file path from the file path table
4.  Depending of if the file path matches the input filename to extract
    1.  If the file path matches the input filename
        1.  Use the file's metadata entry to locate the block offset entry
        2.  Use the block offset entry to locate each block in the archive and read it into memory
        3.  If the file metadata entry indicates the file is compressed, send each block through the decompression interface
        4.  Write out the transformed blocks into the end of an in-memory buffer sequentially
        5.  Break out of iterating the file metadata
    2.  If the file path doesn't match the input filename, then the file is skipped
5.  Depending of if the file was found and its data have been read into memory, write it to disk  
    1.  If the user specified a path, write the contents to that path
    2.  Otherwise use write the contents out to current working directory using the filename segment

### How are file path lookup's resolved when multiple archives are mounted?

In version 1, file paths will be searched from the list of mounted archives in order from last mounted to first.  
To facilitate fast queries, the approach that could be done is to retrieve the file path information for each file within the archive  
and add it to a tree or mapping of file path to a value stack of mounted archive index + file path index.

### Technical API

Below is a sample of how the C++ API could look for the new Archive System.  
This section will be updated as the API is fleshed out more

#### Archive Format Data Layout

<details>
  <summary style="font-weight: bold">Archive Format Data Layout</summary>

```c++
namespace Archive
{
    inline namespace literals
    {
        constexpr AZ::u64 operator"" _kib(AZ::u64 value);
        constexpr AZ::u64 operator"" _mib(AZ::u64 value);
        constexpr AZ::u64 operator"" _gib(AZ::u64 value);
    }

    //! Represents the default block size for the Archive format
    //! It will be 2 MiB until their is data that proves
    //! that a different block size is ideal
    constexpr AZ::u64 ArchiveDefaultBlockSize = 2  * (1 << 20);
    //! The alignment of blocks within an archive file
    //! It defaults to 512 bytes
    constexpr AZ::u64 ArchiveDefaultBlockAlignment = 512;

    //! Sentinel which indicates the last entry in the deleted File Path Index list
    constexpr AZ::u32 DeletedPathIndexSentinel = AZStd::numeric_limits<AZ::u32>::max();

    //! Sentinel which indicates the value written to the last block to indicate
    //! there are no further deleted blocks afterwards
    constexpr AZ::u64 DeletedBlockOffsetSentinel = AZStd::numeric_limits<AZ::u64>::max();

    //! Fixed size Header struct for the Archive format
    //! This suitable for directly reading the archive header into
    struct ArchiveHeaderSection
    {
        ArchiveHeaderSection();

        //! O3DE only runs on little endian machines
        //! Therefore the bytes are added in little endian order
        //! The Magic Identifier for the archive format is "O3AR" for O3DE Archive
        //! offset = 0
        const AZ::u32 m_magicBytes{ 'O' | ('3' << 8) | ('A' << 16) | ('R' << 24) };

        //! Version number of archive format. Supports up to 2^16 revisions per entry
        //! offset = 4
        AZ::u16 m_minorVersion{};
        AZ::u16 m_majorVersion{};
        AZ::u16 m_revision{};

        //! reserved for future memory configurations
        //! default layout is 2Mib blocks with 512 byte borders
        //! offset = 10
        AZ::u16 m_layout{};

        //! Represents the The number of files stored within the Archive
        //! Caps out at (2^25) or ~33 million files that can be represented
        //! offset = 12
        AZ::u32 m_fileCount{};

        //! The 64-bit offset from the start of the Archive File to the Table of Contents
        //! offset = 16
        AZ::u64 m_tocOffset{};

        //! Compressed Size of the Table of Contents
        //! Max size is 512MiB or 2^29 bytes
        //! The TOC offset + TOC offset is equal to total size of the Archive file
        //! offset = 24
        AZ::u32 m_tocCompressedSize : 29;
        //! Compression algorithm used for the Table of Contents
        //! The maximum 3-bit value of 7 is reserved for uncompressed
        //! Other values count as a offset in the Compression Algorithm ID table
        AZ::u32 m_tocCompressionAlgoIndex : 3;

        //! Uncompressed size of the Table of Contents File Metadata section.
        //! offset = 28
        AZ::u32 m_tocFileMetadataTableUncompressedSize{};
        //! Uncompressed size of the Table of Contents File Path index
        //! The File Path index is used to lookup the location for a file path
        //! within the archive
        //! offset = 32
        AZ::u32 m_tocPathIndexTableUncompressedSize{};
        //! Uncompressed size of the Table of Contents File Path section
        //! It contains a blob of FilePaths without any null-termination
        //! The File Path Index entries are used to look up a file path
        //! through using the path offset + size entry
        //! offset = 36
        AZ::u32 m_tocPathTableUncompressedSize{};
        //! Uncompressed size of the Table of Contents File Block section
        //! Contains compressed sizes of individual blocks of a file
        //! In Archive V1 layout the block size is 2MiB
        //! offset = 40
        AZ::u32 m_tocBlockTableSize{};

        //! Threshold value represents the cap on the size a block after it has been
        //! sent through the compression step to determine if it should be stored compressed
        //!
        //! Due to block size defaulting to 2MiB, any blocks that are larger than 2_mib
        //! after compression will be stored uncompressed.
        //! So the maximum limit of this value is the Block Size
        //! offset = 44
        AZ::u32 m_compressionThreshold{ static_cast<AZ::u32>(ArchiveDefaultBlockSize) };

        //! Stores 32-bit IDS of up to 7 compression algorithms that this archive can use
        //! offset = 48
        AZStd::array<Compression::CompressionAlgorithmId, 7> m_compressionAlgorithmsIds;

        //! Offset from the beginning of the File Path index table
        //! Where the first deleted block is located
        //! offset = 76
        AZ::u32 m_firstDeletedFileIndex{ DeletedPathIndexSentinel };

        //! Offset from the beginning of the file block section to the first
        //! deleted block.
        //! The first 8 bytes of each deleted block will contain the offset to the next deleted block
        //! or 0xffff'ffff'ffff'ffff if this is the last deleted block
        //! offset = 80
        AZ::u64 m_firstDeletedBlockOffset{ DeletedBlockOffsetSentinel };

        //! total offset = 88

        //! Max FileCount
        static constexpr AZ::u32 MaxFileCount{(1 << 25) - 1 };
    };

    static_assert(sizeof(ArchiveHeaderSection) == 88, "Archive Header section should be less than 88 bytes per spec version 1");
    static_assert(sizeof(ArchiveHeaderSection) <= ArchiveDefaultBlockAlignment, "Archive Header section should be less than 512 bytes");

    //! Represents an entry of a single file within the Archive
    struct ArchiveFileMetadataSection
    {
        ArchiveFileMetadataSection();

        //! represents the file after it has been uncompressed on disk
        //! Can represent a file up to 2^35 = 32GiB
        AZ::u64 m_uncompressedSize : 35; // 35-bits
        //! compressed files are stored aligned on 512-byte sectors
        //! Therefore this can represent size up to 35-bits as well
        AZ::u64 m_compressedSize : 26; // 61-bits
        //! Stores an index into Compression ID table to indicate
        //! the compression algorithm the file uses or UncompressedAlgorithmIndex
        //! if the value is set to UncompressedAlgorithmIndex,
        //! the block Table Index is not used
        AZ::u64 m_compressionAlgoIndex : 3; // 64-bits

        //! Index for the first block which contains compressed data for this file
        //! As it is 25 bits, up to 2^25 ~ 33.55 Million blocks can be referenced
        //! If the compressionAlgorithm index is UncompressedAlgorithmIndex
        //! then the file is uncompressed in m_offset in a contiguous block
        //! that is 512 byte aligned
        AZ::u64 m_blockTableIndex : 25; // 25-bits
        //! Offset within the archive where the file actually starts
        //! Due to files within the archive being aligned on 512-byte boundaries
        //! This can represent a offset of up to (39 + 9) bits or 2^48 = 256TiB
        //! The actual cap for Archive V1 layout is around 64TiB, since the m_blockTable can only
        //! represent 2^25 "2 MiB" blocks, which is (2^25 * 2^21) = 2^46 = 64TiB
        AZ::u64 m_offset : 39; // 64-bits
    };

    //! Views an entry of a single File Path Index when the File Path Table of the Archive TOC
    struct ArchiveFilePathIndexSection
    {
        ArchiveFilePathIndexSection();

        //! Deleted flag to indicates if the file has been deleted from the archive
        bool m_deleted{};
        //! Because the previous entry was a bool there is 1 byte of padding here before the size

        //! Size of the number of bytes until the end of the File Path entry
        //! Cap is 16-bits to allow relative paths with sizes up to 2^16
        AZ::u16 m_size{};

        //! Offset from the beginning of the File Path Table to the start of Archive File Path
        AZ::u32 m_offset{};
    };


    //! Block lines are made up of 3 blocks at a time
    //! This is used when a file uncompressed size is < 18 MiB
    struct ArchiveBlockLine
    {
        ArchiveBlockLine();

        //! Represents the compressed size of the first 2 MiB block in a block line
        AZ::u64 m_block1 : 21;
        //! Represents the compressed size of the middle 2 MiB block in a block line
        AZ::u64 m_block2 : 21;
        //! Represents the compressed size of the last 2 MiB block in a block line
        AZ::u64 m_block3 : 21;
        //! 1 if the block is used
        AZ::u64 m_blockUsed : 1;
    };

    //! Block line to represents the compressed size of a file in blocks
    //! when a file uncompressed size is >=18 MiB
    struct ArchiveBlockLineJump
    {
        ArchiveBlockLineJump();

        //! 16 bit entry which is used to skip the next 8 blocks
        //! by storing the next 8 block total size within the 16-bits
        //! As blocks are 512-byte aligned, a size of up to 25-bits can be represented
        //! 2^25 = 32 MiB > 18 MiB, therefore jumps of 18MiB can be represented
        AZ::u64 m_blockJump : 16;
        //! Represents the compressed size(non-aligned) of the first block in the block line
        //! containing the jump table
        AZ::u64 m_block1 : 21;
        //! Represents the compressed size(non-aligned) of the last block in the block line
        //! containing the jump table
        AZ::u64 m_block2 : 21;
        //! 1 if the block is used
        AZ::u64 m_blockUsed : 1;
    };

    static_assert(sizeof(ArchiveBlockLine) == sizeof(ArchiveBlockLineJump),
        "The Non-Jump Block Line and Jump Block line must be the same size");


    //! Union which can store either a block line without a jump entry
    //! or block line with a jump entry
    union ArchiveBlockLineSection
    {
        ArchiveBlockLineSection();

        //! A block line containing entries for up to 3 2MiB block
        //! It will be the only type used for files with a total uncompressed size < 18 MiB
        ArchiveBlockLine m_blockLine{};
        //! A block containing a 16-bit jump entry which is used to store the total
        //! compressed size of the next 8-blocks
        //! When the remaining uncompressed size >= 18 MiB, a block with a jump entry
        //! will exist for every 3 block lines until the remaining uncompressed size
        //! is < 18 MiB
        ArchiveBlockLineJump m_blockLineWithJump;
    };

    //! Structure storing data from the Table of Contents at the end
    //! of the archive file
    struct ArchiveTableOfContents
    {
        //! The Archive Header is either uncompressed
        //! or compressed based on the compression algorithm index
        //! set within the Archive Header

        //! Pointer to the beginning of the Archive File Metadata Table
        //! It's length is based on the file count value in the Archive Header Section
        ArchiveFileMetadataSection* m_fileMetadataTable{};
        //! Pointer to the beginning of the Archive File Path Index Table
        //! It's length is based on the file count value in the Archive Header Section
        ArchiveFilePathIndexSection* m_filePathIndexTable{};

        //! ArchiveFilePathTable is a view into a blob of file paths
        using ArchiveFilePathTable = AZStd::span<AZ::IO::PathView>;
        ArchiveFilePathTable m_filePathBlob;

        //! pointer to block offset table which stores the compressed size of all blocks within the archive
        ArchiveBlockLineSection* m_archiveBlockTable{};

        //! Aligns the fileCount up to the next even value, so that
        //! it can maintain an 8-byte alignment
        static constexpr size_t AlignFileCount(size_t fileCount);
    };

} // namespace Archive
```
</details>
<br/>

#### Decompression Interface API  

There will be need to be a decompression interfaces that compression gem scan register their decompression functions with it

<details>
  <summary style="font-weight: bold">Decompression Interface API</summary>

```c++
namespace Compression
{
    enum class DecompressionResult : AZ::u8
    {
        PendingStart,
        Started,
        Complete,
        Failed,
    };

    using DecompressionResultString = AZStd::fixed_string<512>;

    //! Structure which can be used to supply custom options
    //! to the decompression interface DeompressBlock function
    //! The expected use is that the derived IDeCompressionInterface class
    //! implements the Decompression struct and additional fields for specifying decompression options
    //! When the derived compressed interface classes in their DecompressBlock function
    //! would use a call to AZ rtti cast to cast the struct to the correct type derived type
    //! `azrtti_cast<derived-decompression-options*>(&decompressionOptions);
    struct DecompressionOptions
    {
        AZ_TYPE_INFO_WITH_NAME_DECL(DecompressionOptions);
        AZ_RTTI_NO_TYPE_INFO_DECL();
        virtual ~DecompressionOptions();
    };

    struct DecompressionOutcome
    {
        explicit constexpr operator bool() const;
        //! Stores result code of whether the operation succeeded
        DecompressionResult m_result{ DecompressionResult::PendingStart };
        //! Stores any error messages associated with a failure result
        DecompressionResultString m_resultString;
    };

    struct DecompressionResultData
    {
        //! Returns a boolean true if decompression has succeeded
        explicit constexpr operator bool() const;

        //! Retrieves the uncompressed size of the decompression data
        //! from the span
        AZ::u64 GetUncompressedByteCount() const;

        //! Retrieves the memory address where the uncompressed data
        //! is stored
        AZStd::byte* GetUncompressedByteData() const;

        //! Will be set to the memory address of the uncompressed buffer
        //! The size of the span will be set to actual uncompressed size
        AZStd::span<AZStd::byte> m_uncompressedBuffer;

        //! Outcome containing result of the decompression operation
        DecompressionOutcome m_decompressionOutcome;
    };

    struct IDecompressionInterface
    {
        virtual ~IDecompressionInterface() = default;

        //! Retrieves the 32-bit compression algorithm ID associated with this interface
        virtual CompressionAlgorithmId GetCompressionAlgorithmId() const = 0;

        //! Decompresses the input compressed data into the uncompressed buffer
        //! Both parameters are specified as spans which encapsulates the contiguous buffer and it's size.
        //! @param uncompressedBuffer destination buffer where uncompress output will be stored
        //! @param compressedData source buffer containing compressed data to decompress
        //! @param decompressionOptions that can be provided to the Compressor
        //! @return a DecompressionResultData instance to indicate if compression operation has succeeded
        [[nodiscard]] virtual DecompressionResultData DecompressBlock(
            AZStd::span<AZStd::byte> decompressionBuffer, const AZStd::span<const AZStd::byte>& compressedData,
            const DecompressionOptions& decompressionOptions = {}) const = 0;
    };

    class DecompressionRegistrarInterface
    {
    public:
        AZ_TYPE_INFO_WITH_NAME_DECL(DecompressionRegistrarInterface);
        AZ_RTTI_NO_TYPE_INFO_DECL();
        virtual ~DecompressionRegistrarInterface() = default;

        //! Callback function that is invoked for every registered decompression interface
        //! return true to indicate that visitation of decompression interfaces should continue
        //! returning false halts iteration
        using VisitDecompressionInterfaceCallback = AZStd::function<bool(IDecompressionInterface&)>;
        //! Invokes the supplied visitor for each register decompression interface that is non-nullptr
        virtual void VisitDecompressionInterfaces(const VisitDecompressionInterfaceCallback&) const = 0;


        //! Registers decompression interface and takes ownership of it if registration is successful
        //! @param compressionAlgorithmId Unique id to associate with decompression interface
        //! @param decompressionInterface decompression interface to register
        //! @return Success outcome if the decompression interface was successfully registered
        //! Otherwise, a failure outcome with the decompression interface is forward back to the caller
        virtual AZ::Outcome<void, AZStd::unique_ptr<IDecompressionInterface>> RegisterDecompressionInterface(
            CompressionAlgorithmId compressionAlgorithmId, AZStd::unique_ptr<IDecompressionInterface> decompressionInterface) = 0;


        //! Registers decompression interface, but does not take ownership of it
        //! If a decompression interface with a CompressionAlgorithmId is registered that
        //! matches the input decompression interface, then registration does not occur
        //!
        //! Registers decompression interface, but does not take ownership of it
        //! @param compressionAlgorithmId Unique id to associate with decompression interface
        //! @param decompressionInterface decompression interface to register
        //! @return true if the ICompressionInterface was successfully registered
        virtual bool RegisterDecompressionInterface(CompressionAlgorithmId compressionAlgorithmId, IDecompressionInterface& decompressionInterface) = 0;

        //! Unregisters the decompression interface with the specified id
        //!
        //! @param decompressionAlgorithmId unique Id that identifies the decompression interface
        //! @return true if the unregistration is successful
        virtual bool UnregisterDecompressionInterface(CompressionAlgorithmId compressionAlgorithmId) = 0;

        //! Queries the decompression interface with the decompression algorithm Id
        //! @param compressionAlgorithmId unique Id of decompression interface to query
        //! @return pointer to the decompression interface or nullptr if not found
        [[nodiscard]] virtual IDecompressionInterface* FindDecompressionInterface(CompressionAlgorithmId compressionAlgorithmId) const = 0;


        //! Return true if there is an decompression interface registered with the specified id
        //! @param compressionAlgorithmId CompressionAlgorithmId to determine if an decompression interface is registered
        //! @return bool indicating if there is an decompression interface with the id registered
        [[nodiscard]] virtual bool IsRegistered(CompressionAlgorithmId compressionAlgorithmId) const = 0;
    };

    using DecompressionRegistrar = AZ::Interface<DecompressionRegistrarInterface>;

} // namespace Compression
```
</details>
<br/>

#### Compression Interface API

For tooling gems and application such as the AssetBundler, there will be a needed to access the compression functions a compression gem can provide

<details>
  <summary style="font-weight: bold">Compression Interface API</summary>

```c++
namespace Compression
{
    enum class CompressionResult : AZ::u8
    {
        PendingStart,
        Started,
        Complete,
        Failed,
    };

    using CompressionResultString = AZStd::fixed_string<256>;

    //! Structure which can be used to supply custom options
    //! to the compression interface CompressBlock function
    //! The expected use is that the derived ICompressionInterface class
    //! implements the CompressionOptions struct and additional fields for specifying compression options
    //! When the derived compression interface classes in their CompressBlock function
    //! would use a call to AZ rtti cast to cast the struct to the correct type derived type
    //! `azrtti_cast<derived-compression-options*>(&compressionOptions);
    struct CompressionOptions
    {
        AZ_TYPE_INFO_WITH_NAME_DECL(CompressionOptions);
        AZ_RTTI_NO_TYPE_INFO_DECL();
        virtual ~CompressionOptions();
    };

    struct CompressionOutcome
    {
        explicit constexpr operator bool() const;
        //! Stores result code of whether the operation succeeded
        CompressionResult m_result{ CompressionResult::PendingStart };
        //! Stores any error messages associated with a failure result
        CompressionResultString m_resultString;
    };

    struct CompressionResultData
    {
        //! Returns a boolean true if compression has completed
        explicit constexpr operator bool() const;

        //! Retrieves the compressed byte size
        AZ::u64 GetCompressedByteCount() const;

        //! Retrieves the memory address of the compressed data
        AZStd::byte* GetCompressedByteData() const;

        //! Will be set to the memory address of the compressed buffer
        //! supplied to the compression interface CompressBlock command
        //! The size of the span will be set to actual uncompressed size
        AZStd::span<AZStd::byte> m_compressedBuffer;

        //! Outcome containing result of the compression operation
        CompressionOutcome m_compressionOutcome;
    };

    struct ICompressionInterface
    {
        virtual ~ICompressionInterface() = default;

        //! Retrieves the 32-bit compression algorithm ID associated with this interface
        virtual CompressionAlgorithmId GetCompressionAlgorithmId() const = 0;
        //! Compresses the uncompressed data into the compressed buffer
        //! @param compressedBuffer destination buffer where compressed output will be stored to
        //! @param uncompressedData source buffer containing uncompressed content
        //! @param compressOptions that can be provided to the Compressor
        //! @return a CompressionResultData instance to indicate if compression operation has succeeded
        [[nodiscard]] virtual CompressionResultData CompressBlock(
            AZStd::span<AZStd::byte> compressionBuffer, const AZStd::span<const AZStd::byte>& uncompressedData,
            const CompressionOptions& compressionOptions = {}) const = 0;

        //! Returns the upper bound on compressed size given the uncompressed buffer size
        //! Can be used to allocate a destination buffer that can fit the compressed content
        //! @param uncompressedBufferSize size of uncompressed data
        //! @return worst case(upper bound) size that is needed to store compressed data for a given uncompressed size
        [[nodiscard]] virtual size_t CompressBound(size_t uncompressedBufferSize) const = 0;
    };

    class CompressionRegistrarInterface
    {
    public:
        AZ_TYPE_INFO_WITH_NAME_DECL(CompressionRegistrarInterface);
        AZ_RTTI_NO_TYPE_INFO_DECL();
        virtual ~CompressionRegistrarInterface() = default;

        //! Callback function that is invoked for every registered compression interface
        //! return true to indicate that visitation of compression interfaces should continue
        //! returning false halts iteration
        using VisitCompressionInterfaceCallback = AZStd::function<bool(ICompressionInterface&)>;
        //! Invokes the supplied visitor for each register compression interface that is non-nullptr
        virtual void VisitCompressionInterfaces(const VisitCompressionInterfaceCallback&) const = 0;

        //! Registers compression interface and takes ownership of it if registration is successful
        //! @param compressionAlgorithmId Unique id to associate with compression interface
        //! @param compressionInterface compression interface to register
        //! @return Success outcome if the compression interface was successfully registered
        //! Otherwise, a failure outcome with the compression interface is forward back to the caller
        virtual AZ::Outcome<void, AZStd::unique_ptr<ICompressionInterface>> RegisterCompressionInterface(
            CompressionAlgorithmId compressionAlgorithmId, AZStd::unique_ptr<ICompressionInterface> compressionInterface) = 0;


        //! Registers compression interface, but does not take ownership of it
        //! If a compression interface with a CompressionAlgorithmId is registered that
        //! matches the input compression interface, then registration does not occur
        //!
        //! Registers compression interface, but does not take ownership of it
        //! @param compressionAlgorithmId Unique id to associate with compression interface
        //! @param compressionInterface compression interface to register
        //! @return true if the ICompressionInterface was successfully registered
        virtual bool RegisterCompressionInterface(CompressionAlgorithmId compressionAlgorithmId, ICompressionInterface& compressionInterface) = 0;

        //! Unregisters the compression interface with the specified id
        //!
        //! @param compressionAlgorithmId unique Id that identifies the compression interface
        //! @return true if the unregistration is successful
        virtual bool UnregisterCompressionInterface(CompressionAlgorithmId compressionAlgorithmId) = 0;

        //! Queries the compression interface with the compression algorithm Id
        //! @param compressionAlgorithmId unique Id of compression interface to query
        //! @return pointer to the compression interface or nullptr if not found
        [[nodiscard]] virtual ICompressionInterface* FindCompressionInterface(CompressionAlgorithmId compressionAlgorithmId) const = 0;


        //! Return true if there is an compression interface registered with the specified id
        //! @param compressionAlgorithmId CompressionAlgorithmId to determine if an compression interface is registered
        //! @return bool indicating if there is an compression interface with the id registered
        [[nodiscard]] virtual bool IsRegistered(CompressionAlgorithmId compressionAlgorithmId) const = 0;
    };

    using CompressionRegistrar = AZ::Interface<CompressionRegistrarInterface>;

} // namespace Compression
```
</details>
<br/>

#### Archive In-Memory API

The In-Memory API structures contains the structures used by the Archive Reader and Archive Writer in a running application to store data about the archive as it is being processed.  
The structures differ from the data layout of the Archive file on disk are made with ease of use in the runtime.  

<details>
  <summary style="font-weight: bold">Archive In-Memory API</summary>

```c++
namespace Archive
{
    //! Token that can be used to identify a file within an archive
    enum class ArchiveFileToken : AZ::u64 {};

    static constexpr ArchiveFileToken InvalidArchiveFileToken = static_cast<ArchiveFileToken>(AZ::u64(0));


    // ! Returns metadata about an archived file
    struct ArchiveFileMetadata
    {
        //! Relative File path which represents the file in the Archive
        AZ::IO::PathView m_filePath{};
        //! Offset to the first block of the Archive File on disk
        //! If the compression algorithm = Uncompressed
        //! this represents a single contiguous block of file data
        AZ::u64 m_offset{};
        //! Uncompressed size of the file
        AZ::u64 m_uncompressedSize{};
        //! This will be 0 if the compression algorithm = Uncompressed
        AZ::u64 m_compressedSize{};
        //! The compression algorithm used to compress this file
        //! the archive
        Compression::CompressionAlgorithmId m_compressionAlgorithm{ Compression::Uncompressed };
    };
} // namespace Archive
```

</details>
<br/>

#### Archive Reader API  

The Archive Reader will be able to read and parse the Archive header from a single archive file.  
The "Mount" function will read the Archive header into the Reader from an input stream.  
The "Unmount" function will drop that Archive header data  

Archive Reader does not maintain File ownership

The Archive Reader does not store a SystemFile or a GenericStream that will be able to read the archive data.  
The API supports supplying a GenericStream, which is used by to read the Archive Header and Table of Contents data from

<details>
  <summary style="font-weight: bold">Archive Reader API</summary>

```c++
enum class ArchiveReaderMountResult : u8
{
    Mounted,
    NotMounted,
};
 
enum class ArchiveReadResult : u8
{
    Started,
    Complete,
    NotStarted,
    Failed,
};
 
// ! Returns metadata about an archived file
struct ArchiveFileMetadata
{
    //! Relative File path which represents the file in the Archive
    AZ::IO::PathView m_filePath{};
    //! Offset to the first block of the Archive File on disk
    //! If the compression algorithm = Uncompressed
    //! this represents a single contiguous block of file data
    AZ::u64 m_offset{};
    //! Unompressed size of the file
    AZ::u64 m_uncompressedSize{};
    //! This will be 0 if the compression algorithm = Uncompressed
    AZ::u64 m_compressedSize{};
    //! The compression algorithm used to compress this file
    //! the archive
    CompressionAlgorithmId m_compressionAlgorithm{ Uncompressed };
};
 
struct ArchiveReaderSettings
{};
 
 
class ArchiveReader
{
public:
    AZ_RTTI(ArchiveReader, "{DAEEC567-A87A-4CD8-974B-8C1B0446F9FB}");
    AZ_CLASS_ALLOCATOR(ArchiveReader, 0, AZ::SystemAllocator);
 
    ArchiveReader() = default;
    ~ArchiveReader();
 
    explicit ArchiveReader(const ArchiveReaderSettings& readerSettings);
 
    //! Initializes the ArchiveReader using the ArchiveReaderSetings
    //! and reads the archive header from the supplied stream
    ArchiveReader(AZ::IO::GenericStream& inputStream, const ArchiveReaderSettings& readerSettings = {});
 
    //! Reads the ArchiveHeader from the input stream
    ArchiveReaderMountResult Mount(AZ::IO::GenericStream& inputStream);
 
    //! Returns true if archive file contains a valid header
    bool IsMounted() const;
 
    //! Unmounts the archive file
    //! This drops the loaded header data from the Archive
    void Unmount();
 
    //! Searches for a relativePath
    //! @return A token that identifies the Archive file if it exist
    //! if the with the specified path doesn't exist ArchiveFileToken::InvalidToken is returned
    ArchiveFileMetadata GetFileMetadata(AZ::IO::PathView relativePath);
    ArchiveFileMetadata GetFileMetadata(ArchiveFileToken relativePathToken);
 
private:
    //! Reads the Archive Reader Header from the stream into this ArchiveReader
    bool ReadHeader(AZ::IO::GenericStream& inputStream) const;
 
    ArchiveReaderSettings m_archiveReaderSettings;
 
    AZStd::unique_ptr<ArchiveHeader> m_header;
    AZStd::unique_ptr<ArchiveTableOfContents> m_toc;
};
```

</details>
<br/>

#### Archive Writer API

The Archive Writer interface will only be available to Tool applications and .Tools variants of gems to allow creation of Archive files using the new format


<details>
  <summary style="font-weight: bold">Archive Writer API</summary>

```c++
namespace Archive
{
    //! Stores settings to configure how Archive Writer Settings
    struct ArchiveWriterSettings
    {
        ArchiveWriterSettings() = default;

        //! Initial header used for writing files to the archive
        //! It contains the supported compression format and the offset to the table of contents entries
        //! This can be used to update an existing archive when that Archive header is supplied
        ArchiveHeaderSection m_initArchiveHeader{};
        //! Initial Table of contents to be used by the archive writer
        //! This is useful to set when updating an existing archive file using the Archive Writer
        //! The Table of contents provides access to the File metadata, file path names and block offset table
        //! These tables contain all the information necessary to locate existing files within an archive
        ArchiveTableOfContents m_initArchiveTOC{};
    };

    //! Specifies settings to use when retrieving the metadata
    //! about files within the archive
    struct ArchiveWriterMetadataSettings
    {
        //! Output total file count
        bool m_writeFileCount{ true };
        //! Outputs the relative file paths
        bool m_writeFilePaths{ true };
        //! Outputs the offsets of files within the archive
        bool m_writeFileOffsets{ true };
        //! Outputs the sizes of file as they are stored inside of an archive
        //! as well as the compression algorithm used for files
        //! This will include both uncompressed and compressed sizes
        bool m_writeFileSizesAndCompression{ true };
    };

    enum class ArchiveWriterFileMode : bool
    {
        AddNew,
        AddNewOrUpdateExisting,
    };

    enum class ArchiveFilePathCase : AZ::u8
    {
        // Lowercase the file path when adding to the Archive
        Lowercase,
        // Uppercase the file path when adding to the Archive
        Uppercase,
        // Maintain the current file path case when adding to the Archive
        Keep
    };

    //! Settings for controlling how an individual file is added to an archive.
    //! It supports specification of the compression algorithm,
    //! the relative path it should be in the archive file located at within the archive,
    //! whether to allow updating an existing archive file at the same path, etc...
    //! NOTE: The relative file path will be lowercased by default based on the
    //! ArchiveFileCase enum
    //! This due to the Archiving System supporting both case-preserving(Windows, MacOS)
    //! and case-sensitive systems such as Windows
    struct ArchiveWriterFileSettings
    {
        AZ::IO::Path m_relativeFilePath;
        Compression::CompressionAlgorithmId m_compressionAlgorithm{ Compression::Uncompressed };
        ArchiveWriterFileMode m_fileMode{ ArchiveWriterFileMode::AddNew };
        ArchiveFilePathCase m_fileCase{ ArchiveFilePathCase::Lowercase };
        //! Pointer to a compression options derived struct
        //! This can be used to supply custom compression options
        //! to the compressor the Archive Writer users
        const Compression::CompressionOptions* m_compressionOptions{};
    };

    //! Stores outcome detailing result of operation
    using ResultString = AZStd::fixed_string<512>;
    using ResultOutcome = AZStd::expected<void, ResultString>;

    //! Returns result data around operation of adding a stream of content data
    //! to an archive file
    struct ArchiveAddToFileResult
    {
        //! returns if adding a stream of data to a file within the archive has succeeded
        //! it does by checking that the ArchiveFileToken != InvalidArchiveFileToken
        explicit operator bool() const;

        AZ::IO::PathView m_relativeFilePath;
        ArchiveFileToken m_filePathToken{ InvalidArchiveFileToken };
        Compression::CompressionAlgorithmId m_compressionAlgorithm{ Compression::Uncompressed };
        ResultOutcome m_resultOutcome;
    };

    //! Stores offset information about the Files added to the Archive for write
    struct ArchiveWriterFileMetadata
    {
        AZ::IO::Path m_relativeFilePath;
        Compression::CompressionAlgorithmId m_compressionAlgorithm{ Compression::Uncompressed };
    };
} // namespace Archive
```

</details>
<br/>

File paths are by default lowercased within an archive

As O3DE supports both case-preserving and case-sensitive file systems, the file paths within the archive are lowercased by to support creating an archive on one type of filesystem and reading the archive on another.

For example adding files to an existing archive on NTFS(case-perserving) that was created on an Ext4 filesystem(case-sensitive) needs to be consistent.  
  
This behavior can furthermore prevent relative paths with different cases from being added to the archive.  
Specifying a relative path of "Assets/FOO.txt and "Assets/foo.txt" on an Ext4 system will result in the same relative path of "foo.txt" being added within the archive.

#### Archive Streamer Integration API

In order to integrate the Archive format with the AzCore Streamer System, a new Streamer Stack Entry will need to be added registers

<details>
<summary style="font-weight:bold;">Archive Mounter API</summary>

```c++
//! Represents the mounted index of the within the ArchiveMounter Streamer entry
enum ArchiveMountIndex : u32{};
constexpr ArchiveMountIndex InvalidMountedIndex = static_cast<ArchiveMountIndex>(AZStd::numeric_limits<u32>::max());
 
//! Structure which encapsulates searching for entries among multiple mounted archives
//! The ArchiveMountIndex is a ArchiveMounterStackEntry relative unique value that indentifies it's location
//! within the entry
//! The FilePathIndex represents an the index from the start of File Path Index table corresponding to the mounted archive
struct ArchiveMountedPathIndex
{
    ArchiveMountIndex m_archiveMountIndex{ InvalidMountedIndex };
    u32 m_filePathIndex { DeletedPathIndexSentinel };
};
using ArchiveMountedPathStack = AZStd::stack<ArchiveMountedPathIndex>;
using PathToArchiveMountedPathTable =  AZStd::unordered_map<AZ:IO::Path, ArchiveMountedPathStack>;
 
 
//! Stores configuration settings for the Streamer Stack Entry
//! for the Archive Mounter
struct ArchiveMounterStackSettings
{
    //! Maximum number of block reads
    u32 m_maxNumBlockReads{ AZStd::thread::hardware_concurrency() / 2 };
    //! Maximum number of decompression jobs that the archive will run
    u32 m_maxDecompressionJobs{ AZStd::thread::hardware_concurrency() / 2 };
 
    // Sets a default value for the block read buffer
    u64 m_readScratchBufferSize{ AZ::SizeAlignUp(ArchiveDefaultBlockSize * 0.9, ArchiveDefaultBlockAlignment)
        * AZStd::thread::hardware_concurrency() };
    // Sets a default value for the compression scratch buffer
    u64 m_compressionScratchBufferSize{ SizeAlignUp(ArchiveDefaultBlockSize * 0.9, ArchiveDefaultBlockAlignment)
        * AZStd::thread::hardware_concurrency() };
};
 
struct ArchiveMounterStackConfig final
    : public IStreamerStackConfig
{
    AZ_RTTI(ArchiveMounterConfig, "{2552BDFE-FAE2-4915-8BD0-C91B2830B631}", IStreamerStackConfig);
    AZ_CLASS_ALLOCATOR(ArchiveMounterConfig, AZ::SystemAllocator, 0);
 
    // Will read any default settings overrides from settings registry
    ArchiveMounterStackConfig();
    ~ArchiveMounterStackConfig() override = default;
    AZStd::shared_ptr<StreamStackEntry> AddStreamStackEntry(
        const HardwareInformation& hardware, AZStd::shared_ptr<StreamStackEntry> parent) override;
    static void Reflect(AZ::ReflectContext* context);
 
    //! Settings that will be used to configure the Archive Mounter Stack Entry
    ArchiveMounterStackSettings m_archiveMounterStackSettings;
};
 
//! Entry in the streaming stack that mounts a new fromat Archive file and supports
//! reading file entries from the archive itself
//! It does not handle file decompression, as that will be done via the IDecompression Interface
class ArchiveMounterStackEntry
    : public StreamStackEntry
{
public:
    ArchiveMounterStackEntry();
    explicit ArchiveMounterStackEntry(const ArchiveMounterStackSettings&);
    ~ArchiveMounterStackEntry() override = default;
 
    void PrepareRequest(FileRequest* request) override;
    void QueueRequest(FileRequest* request) override;
    bool ExecuteRequests() override;
 
    void UpdateStatus(Status& status) const override;
    void UpdateCompletionEstimates(AZStd::chrono::system_clock::time_point now, AZStd::vector<FileRequest*>& internalPending,
        StreamerContext::PreparedQueue::iterator pendingBegin, StreamerContext::PreparedQueue::iterator pendingEnd) override;
 
    void CollectStatistics(AZStd::vector<Statistic>& statistics) const override;
 
    //! Path to archive file to mount
    bool Mount(AZ::IO::PathView archivePath);
    //! Path to archive file to unmount
    bool Mount(AZ::IO::PathView archivePath);
 
private:
    AZStd::vector<ArchiveReader> m_mountedArchives;
    AZStd::vector<SystemFile> m_archiveFiles;
    AZStd::vector<AZStd::byte> m_blockReadScratchBuffer;
    AZStd::vector<AZStd::byte> m_compressionScratchBuffer;
    PathToArchiveMountedPathTable m_filePathToArchiveTable;
};
```

</details>
<br/>

#### Streamer API Updates  

Finally a new command will need to be added to the Streamer FileRequest API to support mounting and unmounting an archive

<details>
<summary style="font-weight:bold;">Streamer API updates</summary>

```c++
/* FileRequest.h */
//! mounts or unmounts the provided archive file from the stream stack
struct ArchiveMountData
{
    inline constexpr static IStreamerTypes::Priority s_orderPriority = IStreamerTypes::s_priorityHigh;
    inline constexpr static bool s_failWhenUnhandled = false;
 
    explicit ArchiveMountData(const RequestPath& path);
 
    const RequestPath& m_path;
};
 
struct ArchiveUnmountData
{
    inline constexpr static IStreamerTypes::Priority s_orderPriority = IStreamerTypes::s_priorityHigh;
    inline constexpr static bool s_failWhenUnhandled = false;
 
    explicit ArchiveUnmountData(const RequestPath& path);
 
    const RequestPath& m_path;
};
 
using CommandVariant = AZStd::variant<
    AZStd::monostate,
    ExternalRequestData,
    RequestPathStoreData,
    ReadRequestData,
    ReadData,
    CompressedReadData,
    WaitData,
    FileExistsCheckData,
    FileMetaDataRetrievalData,
    CancelData,
    RescheduleData,
    FlushData,
    FlushAllData,
    CreateDedicatedCacheData,
    DestroyDedicatedCacheData,
    ReportData,
    CustomData,
    ArchiveMountData,
    ArchiveUnmountData>;
 
 
// ...
/* Streamer.h */
/**
 * Data streamer.
 */
class Streamer final
    : public AZ::IO::IStreamer
{
    //
    // Archive Reader
    //
 
    //! Mounts the archive file provided path the streamer stack
    FileRequestPtr MountArchive(AZ::IO::PathView archivePath) override;
    //! Unmounts the archive file at the specified path
    FileRequestPtr UnmountArchive(AZ::IO::PathView archivePath) override;
    // ...
};
```

</details>
<br/>

Scope
----------

### AzCore Streaming System

The Streamer API will need to be updated to support request for Mounting and Unmounting archive files.  
It will also be updated to allow registration of Compression Interfaces in a manner that extending the available compression algorithms through the Gem System

### Gem System

A new core compression gem will be added to support LZ4 compressed content out of the box.  
Optionally a support for Zlib and ZStd content will be added a later.  

### Asset Bundler

The Asset Bundler will need to be updated to create archives using the Archive format.  
The existing commands will also need to be updated to access additional arguments to specify compression settings on either a per file basis or based on a file pattern to ease the ability to specify additional settings as stated in the [Streaming Stack Archive - Tools integration](./rfc-core-2023-04-20-streaming-archive.md#tools-integration) section.

Testing plan
-----------------

In addition to the the Testing Plan mentioned in the [Streaming Stack Archive - Testing plan](./rfc-core-2023-04-20-streaming-archive.md#testing-plan) section, there is the possibility to gather benchmark metrics which measures throughput performances across different disk hardware and speeds(SATA, NVMe, USB, etc...).  
These test could be automated and run as part of the Nightly or Weekly build on Continuous Integration Server such as Jenkins.

Execution Plan (And Dependencies)
--------------------------------------

### Phase 1

The first phase will involve the creation compression interfaces that will be used by AZ Streamer to decompress the archive TOC and individual files  
The compression interfaces will also allow tools such as the AssetBundler to be able compress content added to an archive file.  
This phase will create a Core Gem that wraps the LZ4 3rdParty library, with stretch goals to wrap the Zlib 3rdParty and ZStd 3rd library to provide interaction with those algorithms


| Feature | Description | Required? | Completed |
| --- | --- | --- | --- |
| Add Decompression Interface to Compression Gem | This involves adding the Decompression Interface that Streamer can user to decompress files within an archive | Yes | Yes |
| Add Compression API to Compression Gem Tools and Builder modules only | Provides the API for tools such as the AssetBundler use to compress using the Compression gems. | Yes | Yes |
| Update Streamer SystemComponent to register Decompression Interfaces | Update the Streamer system to store the AZ::Interface needed for registering compression algorithms and  <br/>decompression support | Yes | Yes |
| Create Gem with LZ4 Compression support | Responsible for creating registering the LZ4 Decompression Interface with Streamer.  <br/>Furthermore .Tools variants will be able to create Zlib compressed data through a tools-only write interface<br/><br/>LZ4 Offsets Faster Compression and an order of [magnitude faster decompression](https://github.com/lz4/lz4) than Zlib, while trading  <br/>off compression ratio | Yes | Yes |
| Create Gem with Zlib Compression support | Responsible for creating registering the Zlib Deccompression Interface with Streamer.  <br/>Furthermore .Tools variants will be able to create Zlib compressed data through a tools-only write interfaces. | No | No |
| Create Gem with ZStd Compression support | Registers a Zstd Decompression Interface with streamer.  <br/>Gem .Tools variants will be able to produce Zstd compressed content through a Tools-only write interfaces.  <br/>This is optional, as the one of the tenets as stated in the [Streaming Stack Archive - Tenets](./rfc-core-2023-04-20-streaming-archive.md#tenets) section will be to preference read and decompression speed over disk space.  <br/>The Zstd compression algorithm offers better compression than LZ4, but is offers slower decompression | No | No |
| Add Core Compression Gem to the Project Templates | Involves updating the enable\_gems.cmake within the [Project Templates](https://github.com/o3de/o3de/tree/development/Templates) to activate these Gems.  <br/>Will make sure that new projects out of the box supports creation of archives with Zlib and LZ4 support | Yes | No |
| Add Compressor/Decompressor registration support to Compression Gem | Adds an AZ::Interface to Compression Gem that supports registration of block both compression interfaces and decompression interfaces.  <br/>Gem .Tools variants can register with this interface to allow applications such as the AssetBundler  <br/>to compress files. | Yes | Yes |

### Phase 2

The second phase work involves the actual implementation of the Archiving format.  
Support for creation of archive files and reading them will be added in this phase.  
The AZ Streamer will be extended with support to mount archive files, which would allow individual file content to be read from the archive file and supplied to the Streamer decompression stack entry.  
  
A standalone CLI tool will be created that can read an archive file and list the files within the archive and extract individual files.  
The standalone tool can also make modifications archive file such as a creation of a new archive, adding, removing and updating a file within the archive.

There is an optional task to move the CryPak implementation into a gem in this phase as well time permitting.

| Feature | Description | Required? | Completed? |
| --- | --- | --- | --- |
| Add O3DE Archive Gem | Create a gem where the new Archiving format will be implemented.  <br/>This gem will be expose an AZ Streamer entry through the Settings Registry file that contains the config class to load using the Serialize Context.  <br/>See the [AZ IO Streamer 2.0 Configuration](./rfc-core-2023-04-20-streaming-archive-files/AZ-IO-Streamer-2.0-User-Guide.md#configuration) section on how to configure streamer nodes through the Settings Registry| Yes |  Yes |
| Stamp out the API for the new Archive format | The API for the new Archive Format will be added to multiple new files with AzFramework(or AzCore if there are dependency issues) | Yes | No |
| Implement support for an Archive Writer | The Archive Writer interface will be implemented in the Archive Gem - Tools/Builders module only to allow tooling applications to be able to create the new Archive Format | Yes | No |
| Implement support for the Archive Reader | The Archive Reader interface will be implemented in the Archive Gem as part of the Client moudle.  <br/>This will allow the new Stream Stack Entry API to access functions to query and read files from the archive | Yes | No |
| Add new Stream Stack entry to interface with an Archive Reader | The Stream Stack Entry is needed to interface the new Archive Reader with the Streamer IO stack. | Yes | No |
| Update Streamer with support to Mount/Unmount  Archive files | The [IStreamer API](https://github.com/o3de/o3de/blob/5f5dddf46a8a2b6609c2088e70793e89f5ff48dd/Code/Framework/AzCore/AzCore/IO/IStreamer.h#L30-L47) needs to be updated with first class support for mounting the new Archive Files.  <br/>It will need to support the ability to mount and unmount files.  <br/>Several new request types will be added to the [FileRequest API](https://github.com/o3de/o3de/blob/development/Code/Framework/AzCore/AzCore/IO/Streamer/FileRequest.h#L250-L281) to support the new Mount/Unmount commands | Yes | No |
| Update the [streamer.game.setreg](https://github.com/o3de/o3de/blob/development/Registry/streamer.game.setreg) with the settings  <br/>to create an ArchiveReader Entry in the streamer stack | Streamer leaves initialization of Stack entries to serialized data within the Settings Registry.  <br/>Therefore the streamer.game.setreg will need to be updated to add a stack entry the streamer stack | Yes | No |
| Add Test to validate the Archive Writer and Reader | This will need to be added to AzToolsFramework to validate that the archive format can successfully create archive files and read them | Yes | No |
| Add support to AssetBundler to create new Archives | The Asset Bundler commands will need to be updated to accept arguments to specify the archive file format to use.<br/><br/>It will need to be updated to use the Archive Writer to create new format archives out of the Asset Cache | Yes | No |
| Update AssetBundler GUI to allow users to select  <br/>between creating a CryPak or a new Archive | The Asset Bundler UI would need an UX entry for selecting which type of archives to create when making a bundle | Yes | No |
| Add Test to the AssetBundler to validate creation of new  <br/>format archive files | Add a UnitTest to the AssetBundler to also validate that its commands can create a bundle | Yes | No |
| Standalone CLI Archive Modification and Reading Tool | A standalone CLI application can be created to help users validate the contents of archive files, as well for the existence of individual files within the archive | Yes | No |
| Move the CryPak(AZ::IO::Archive) code from AzFramework to a Gem. | With the archive format, the CryPak format can be moved to a gem, that would allow customers that would like to continue to the use that format.  <br/>This also acts as a pseudo deprecation plan from adding first-party support for CryPak to "second" or third party.  <br/>By being in a gem, it can eventually be moved to external repo if it's use by customers is not high. | No | No |
| Add support for globbing files among the mounted Archives | Add support for a class that can model the a directory structure of the file paths of all mounted archives, which can be used for file globbing.  <br/>A directory structure such as Radix Tree can be used to encode path segments used for matching segments of file paths. | No | No |

### Post-Launch

Benchmark test and metrics will be gathered gathered via Jenkins automation to provide profiling data such as read throughput, decompression speed on host platforms.  
Additional disk hardware will be needed to be profile speed differences.  
Statistics gathering will be supported on non-host platforms as well.

The post launch phase is also the time where a GUI for the standalone tool will be created with collaboration from sig-ui-ux

| Feature | Description | Required? | Completed? |
| --- | --- | --- | --- |
| Add profiling for archive read/decompression throughput | Profiling of archives of various sizes will to determine the read throughput of the Archiving System.  <br/>Profiling should occur on with different drive types and connections, as well as different filesystem formats. | Yes | No |
| Collect profile data for various drive hardware:  <br/>Sata-SSD 6GB/s  <br/>Nvme PCI 2.0  <br/>Nvme PCI 3.0  <br/>Nvme PCI 4.0  <br/>Nvme PCI 5.0 (When released)  <br/>USB 3.1 Gen 2x2  <br/>USB 4 | Profiling will run in an environment where are all CPU, memory, filesystems are uniform.  <br/>This profiling can be done on a single machine that has connections to various drive hardware or on different machines where the CPU, filesystem environment, memory, etc... are of the same spec.<br/><br/>The gathering of profiling data will be limited to Windows only with the potential for gathering data for other platforms as more resources as available | Yes | No |
| Collect profile data data for different CPU and memory  <br/>combinations | This type of profiling should occur on with similar disk hardware and filesystems,  <br/>Data could be used to compare decompression rate and read rate with relation to available cores as well as the memory available on the system | Yes | No |
| (Non-Windows) Add filesystem detection of drive that archive resides on | This involves detecting type of Filesystem the Archive is being read from.  <br/>Be that NTFS, ext4, btrtfs, exFat, etc...  <br/>This will be part of the metrics, to determine how much a given filesystem affects read performance  <br/>  <br/>The Streamer Stack Windows Storage Drive already supports detection of the Filesystem, so this only needed for non-Windows platforms. | No | No |
| Collect profile data for various filesystems:  <br/>NTFS  <br/>ext4  <br/>exFat  <br/>APFS | Optionally data will be collected for different formatted filesystems on ideally the same drive hardware.  <br/>This data can be used to provide information to customers as to what filesystem they can use for maximizing file throughput | No | No |
| Standalone GUI Archive Modification and Reading Tool | A standalone GUI tool that leverages the CLI tool can be created to help uses visually view the contents of files within an Archive | No | No |

### Other Dependencies

#### Asset Bundler Preset Feature

With the modifications to the Asset Bundler as part of the new Archive system work, it may be a good time to implement the Asset Bundler Preset feature within the Asset Bundler.  
This feature would work similar to the [CMake Presets](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html) feature which provides a way to aggregate a shared set of project settings in a json file.  
This would help with easing customer workflows around configuring archive creation options and provide a way to persist these settings to disk.

### Migration Plans (Data, and workflows)

Developers would interface with the new Archive System primarily through the Asset Bundler.  
Options will be available to output an archive in the newer format, than creating a CryPak.

Risks
----------

| Risk | Mitigation |
| --- | --- |
| The new Archive Format is bespoke to O3DE.  <br/>There are no existing tools(7-zip, unzip, WinRar, etc...) that can create or read such format files  <br/>This makes it hard to visually see information about the Archive contents. | CLI tools can be created to validate an archive in general is valid.  <br/>The tool itself can also be used to extract individual files.  <br/>Furthermore tools that generate an Archive, will be able to output |
| Implementations for Console Platforms and Android will need custom implementation of AZ Streamer as well as potentially the new archive system to achieve  <br/>the maximum decompression and read performance those platforms are capable. | The sig-platform group will be engaged to help with console specific implementations if benchmarks  <br/>show suboptimal performance. |
| There are existing Archive formats are mature(15+ years) and can provide O3DE with partial solution that gets the engine X%  to it's goal and doesn't require maintaining of a custom archive format | Research other archive Formats such as ZIP64, XAR, TAR, etc... |

Alternatives
-----------------

### Update CryPak to support the new Archiving Features

An alternative to implementing the new Archive format is to update the existing AZ::IO::Archive classes to support the features presented in the [Streaming Stack Archive](./rfc-core-2023-04-20-streaming-archive.md) document.  
The work would involve adding support for ZIP64 to support archives and individual files above 4GiB.

##### Benefits

*   Existing on the market tools will be able to read and create ZIP/ZIP64 Format files, which makes authoring more convenient, as well debugging easier
*   CryPak already exist and does work as an archive format
*   The Zip Format is more amenable to adding files to an existing archive as the Central Directory Record is at the end of the zip file

##### Downsides

*   Decompression performance cannot be achieved the reaches the read speed limits of current hardware (PCIe 4.0 NVMe drives) which are reaching upwards of read speeds of up to 7200MiB/s
*   As decompression is done on per file basis, inside of CryPak single threaded, there is no way to speed up the operation without moving that logic to out of it
*   The CryPak architecture isn't suited to fit within the Streamer Stack
*   Changing CryPak increases the risk of introducing changes that break existing Pak files.
*   Makes it harder to deprecate existing CryPak functionality.  
    Such as removing the ability to load level files from a Pak, attempting to read
*   The ZIP Format isn't the most layout [efficient](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)  
    Each file payload paired with its file header.
*   The CryPak ZIP implementation is means to an end of supporting an archive format.  
    Since there are already existing [solutions](https://libzip.org/license/) for parsing ZIP files, it would be better to use them instead of maintaining CryPak.
*   The Design of CryPak is something that seems to have grown organically and is not well organized.  
    It interfaces back and forth with the AZ FileIO API to read and write files, query the archive file structure(such as query files via wildcard queries, creating and removing file paths within the archive, resolving file IO aliases, etc...)

### Use ZIP64 via libzip for the Archive format

Instead of updating CryPak to support ZIP64 files, an existing 3rdParty library of [libzip](https://libzip.org/) can be used instead to add ZIP 64 support

#### Benefits

*   Stable library which was release in [2005](https://libzip.org/news/release-0.6.html).
*   Comes with ZIP64 support out of the box
*   A custom archive solution doesn't have to be maintained.
*   License can be used with O3DE; It is 3-clause BSD: [https://libzip.org/license/](https://libzip.org/license/)
*   Can use existing UI and CLI to modify and view zip files
*   OS ZIP Tools are out of the box on all O3DE supported host-platforms(Windows, Linux, Mac)
*   Allows removal of CryPak

#### Downsides

*   The Zip format does not have support for custom compression algorithms that are not DEFLATE based.  
    A compliant implementation only supports [DEFLATE](https://en.wikipedia.org/wiki/ZIP_(file_format)#Standardization) or uncompressed data  
    We need support for custom compression algorithms to take advantage of console hardware decompression support
*   Every file payload has a local file header before it. To consecutive read files sequentially, a seek is required.
*   Format doesn't provide control to how file data is packed within the file.  
    It wouldn't be easy to get file data packed in such a way where it is aligned on a particular boundary(512/4094 byte blocks).  
    Wouldn't allow O3DE to take advantage of reading file data in blocks to support parallel reading decompression

### Use the XAR format via the xar library

The XAR format which stands for [eXtensible Archiver](https://github.com/mackyle/xar) is an archive format that splits an Archive file into three section.  
An archive header, a Table of Contents section and a heap section where unstructured data referenced by the Table of Contents can reside.  
The format spec for it is in the [xar repo](https://github.com/mackyle/xar/wiki/xarformat).

#### Benefits

*   Stable library that has been around since [2007](https://github.com/mackyle/xar/wiki/xarformat/_history).  
    A custom archive solution that doesn't have to be maintained.
*   Supports Custom Metadata in its Table of Contents which is XML  
    This allows O3DE to add metadata to files which use custom compression algorithms, so that console hardware decompression can take advantage of it.
*   The XAR Heap section being unformatted, allows O3DE to layout any file data anyway the engine requires  
    This would allowing aligning files into blocks with 512/4096 byte alignment as well as padding compressed file data to 2MiB block boundaries  
    That would fulfull some of the requirements for the Archive System V1 format.
*   Available on MacOS out of the box
*   Compatible License: 3-Clause BSD [https://github.com/mackyle/xar](https://github.com/mackyle/xar)
*   Can use existing UI and CLI to modify and view zip files

#### Downsides

*   Table of Contents is located at the beginning of the archive file right after the header, causing individual file operations such as add/remove/update to need to relocate the entire XAR heap section.
*   The Table of Contents is XML, which while making the format extensible and able to support any type of custom file metadata, requires that an XML parser is required to parse the TOC which is not a high performance operation when mounting an Archive.
*   XML data is not highly compact, though it is very compressible due to it's format.  
    However the TOC is enforced to be [Zlib compressed](https://github.com/mackyle/xar/wiki/xarformat#the-header)
*   Windows/Linux does XAR tools aren't available with out of the box  
    The tools are easy get to unpack them though  
    On Windows [7-Zip](https://www.7-zip.org/) can be be used to unpack XAR content  
    On Linux distros xar is available in the via the package manager

FAQ
---------

### Why not just update CryPak(AZ::IO::Archive) to support these features?

From the drawbacks listed in the [Alternatives](#alternatives) section, it would take more time to update the existing CryPak system to support some of the new archive format features.  
If anything using libzip over CryPak would be a better alternative?

### What about using the XAR format?

The XAR format TOC of contents is at the beginning of the Archive file.  
If the archive format is desired to allow iterative operations such as file updates, adding or removing a file, then XAR would have to move the entire heap section which contains all of the file data each time.  
The existing [XAR CLI](https://mackyle.github.io/xar/xar.html) command doesn't allow modification of an existing archive either, so if O3DE wanted to support individual file modifications, an additional CLI would need to be added.  

The XML parsing aspect of XAR might not be a huge bottleneck at runtime, but metrics are needed of course.  
Though this is probably a minor point in why XAR isn't a main option  

### Do we have a new name for the new archive solution besides "O3AR"?

No. If you can think of one please speak up.

### Would the new Archive System have any effect on the deprecating AZ FileIO API?

Potentially yes. Moving CryPak support to a gem would remove one more system from the Code that uses the FileIO API.  
Currently any application type that is from AzFramework::Application or deeper creates an FileIO API that interfaces with the Archive system.  
Moving CryPak to a gem would allow moving of the Archive FileIO to a gem as well.  
That would leave the Code libraries with only two FileIO APIs left

*   LocalFileIO - Provides to files on a local disk based file system.  
    The only benefit it provides over SystemFile is the ability to resolve FileIOAliases
*   RemoteFileIO - Provides the Virtual File System(VFS) implementation that the AssetProcessor uses to connection with Launcher applications over a network.  
    It is primarily used for serving assets files from a host machine such as Windows or Linux to a non-host platform such as Android, iOS or consoles

  
Additional Frequently Asked Question from the Streaming Stack 2021 document is provided below

References:  
* [Streaming Stack Archive - FAQ](./rfc-core-2023-04-20-streaming-archive.md#faq)  

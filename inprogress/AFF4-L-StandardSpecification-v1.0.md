# AFF4-L Standard v1.0-ALPHA
## Summary

The AFF4-L an extension to the AFF4 forensic image container (AFF4 Standard v1.0a), focussed on logical imaging of filesystems and files (Windows, Linux, \*OS & Android), and the related scaling issues which come with dealing with logical images containing millions of logical file images. 

AFF4-L adds the following features:
- Standardised metadata lexicon for describing filesystem, file and folder metadata, including concepts such as extended attributes; 
- Efficient storage streams tailored for tiny and small datastreams (supplementing AFF4's existing efficient large datastream);
- Standardised file naming of multi-volume images;
- Differentiation of AFF4 and AFF4-L images by file extension naming convention
- Integrity assurance of container metadata

This document is intended for implementers of the AFF4 Logical (AFF4-L) forensic container, and is to be read as an explanation of the AFF4-L Canonical Reference Images v1.0.
These are available via the AFF4 GitHub [https://github.com/aff4/ReferenceImages/aff4l/](https://github.com/aff4/ReferenceImages/aff4l/) .

Where there is ambiguity or difference between what is stated in this document and the Reference Images, the reference images take precedence. Accordingly, this document may be updated to resolve any such differences.

Readers are encouraged to read the original research papers which proposed the format for  background and terminology [1][2][3][4].

## Document Information

|                |                                                                             |
|----------------|-----------------------------------------------------------------------------|
| Authors        | Bradley L Schatz <blschatz@apple.com>                                       |
| Abstract       | Specification for the AFF4 - Logical Forensic Container Format, Version 1.0 |
| Classification | Public                                                                      |
| Keywords       | AFF4-L, AFF4, Advanced Forensic Format, Forensic Image, Logical Image       |

## License
Copyright (2026), Apple Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Revision history

| Revision | Author       | Section | Date      | Comments      |
|----------|--------------|---------|-----------|---------------|
| 0        | BL Schatz    | All     | Aug 2022  | Initial draft |
| 1        | BL Schatz    | All     | Sept 2026 | Pre-release   |

## Acknowledgements
The authors would like to thank XXX, and the organisers of the DFRWS conference for providing a forum which nurtured the research behind to this work.

## Background

The AFF4-L variant of AFF4 was originally published as a paper [1], and shared as patches to the pyaff4 library [5]. 

More recently, a number of vendors have created logical imaging implementations which are based on AFF4-L, but have diverged in their implementations. 

This standard is intended to provide a common standard for interopability by deprecating proposed features which have not worked well in practice; and providing solutions to newly identified needs.

## 1. Legacy AFF4-L Deprecated Features

In the standard we refer to AFF4-L implementations which do not support this standard as Legacy AFF4-L implementations. Accordingly, the initial AFF4-L paper is also considered a legacy implementation. 

### 1.1 Suspect Path based ARN is deprecated

The original AFF4-L paper (section 3.2) specified a naming scheme for file and folder instances, based on a normalisation of the full path and filename to the file/folder being imaged. The normalisation attempted to preserve the human readability and implicit filesystem heirarchy in the instance identifier.  

In practice, the proposed normalisation scheme proved brittle. Defining a set of normalizations rules was overly complex given the variability of character sets used in deployed filesystems. This standard deprecates the naming proposal.

Implementations MUST use ARN's to name AFF4 objects, and use the aff4:fileName\* and aff4:originalPathName\* properties to store the path and name of referred to files and folders. 

### 1.2 ARN to Zip segment escaping is deprecated

In the AFF4 Standard v1.0, the rules regarding the forerunner concept of ARN's are outlined in section 5.2.

In AFF4-L, the prefix component containing "://" is no longer escaped. 

## 2. GUID Based ARN

AFF4-L uses the term AFF4 Resource Name (ARN) to refer to resource names given to AFF4 objects. An ARN is an Internationalized Resource Identifier (IRI) (an extension of an URI) with the existing scheme “aff4://”.

A GUID based ARN in constructed as follows:
```
AFF4-ARN = "aff4://" object-guid-part [ "/" ] [ extensible-part ]
```
The object-guid-part is a lower case UUID/GUID. The extensible part may be any string such that the ARN still conforms to the IRI standard. 

## 3. Container Versioning
AFF4-L versioning extends the AFF4 Standard v1.0 container versioning scheme. For AFF4-L Standard v1.0, the Major is 2, Minor is 1. Below is an example:

```
major=2\n
minor=1\n
tool=pyaff4 0.9\n
```

## 4. Information Model

### 4.1 Lexicon synonyms

The RDF namespace to be used for newly introduced RDF lexicon items introduced in implementations of the AFF4-L Standard v1.0 is http://aff4.org/Schema/2022/# , shorthand prefixed as “aff4l”. 

These supplement the lexicon introduced in the AFF4 Standard v1.0.

The naming convention for properties and classes is unchanged from AFF4 Standard v1.0.

Compliant implementations MUST use the correct namespace as defined in the relevant AFF4/AFF4-L standard when writing. When reading, compliant implementations MAY additionally honour either namespace prefix in support of backwards compatibility. 

### 4.2 Base Classes

The following classes supplement those defined in AFF4 Standard v1.0. 

| Canonical Lexicon Name | Purpose                                                                                    | 
|------------------------|--------------------------------------------------------------------------------------------|
| FileSubStream          | The non-primary data stream of a file (i.e. an Alternate Data Stream or Extended Attribute |
| FileImage              | Logical image of file                                                                      |
| Folder                 | Represents a file system folder                                                            |
| FileExtendedAttribute  | MacOS ExtendedAttribute |
| LogicalAcquisitionTask | Represents the logical acquisition |

### 4.3 Base Properties

| Name                | Context                | Purpose                                                                                                            |
|---------------------|------------------------|--------------------------------------------------------------------------------------------------------------------|
| fileMode            | FileImage & Folder     | Unix file mode, as integer.                                                                                        |
| fileName            | FileImage & Folder     | The normalised name of the file.                                                                                   |
| fileNameRaw         | FileImage & Folder     | The name of the file stored Base64 as read from disk. Only used when the file name cannot be UTF-8 encoded.        |
| originalPathName    | FileImage & Folder     | The normalised full path and file name of the file.                                                                |
| originalPathNameRaw | FileImage & Folder     | The full path and file name stored Base64 as read from disk. Only used when the file name cannot be UTF-8 encoded. |
| birthTime           | FileImage & Folder     | Time and date file was created.                                                                                    |
| lastAccessed        | FileImage & Folder     | "Last Accessed" time and date of file.                                                                             |
| lastWritten         | FileImage & Folder     | Time and date file was last modified.                                                                              |
| recordChanged       | FileImage & Folder     | Time and date file was last modified.                                                                              |
| alternateDataStream | FileImage & Folder     | Reference to FileSubStream (NTFS)                                                                                  |
| extendedAttribute   | FileImage & Folder     | Reference to FileSubStream (HPFS, APFS)                                                                            |
| name                | FileSubStream          | Substream name                                                                                                     |
| value               | FileSubStream          | Content of substream, as xsd:base64Binary                                                                          |
| hash                | FileImage              | The hash of the datastream.                                                                                        |
| filesystemRoot      | LogicalAcquisitionTask | The root folder of a particular tree of acquired files and folders                                                 |
| pathSeparator       | LogicalAcquisitionTask | The separator character in use for the current logical acquisition                                                 |
| name                | FileExtendedAttribute  | The name of an extended attribute                                                                                  |
| value               | FileExtendedAttribute  | The base64 encoded data stream of an extended attribute                                                            |

### 4.4 Additional datatypes

The following additional datatypes are supported for digest properties.

| Name            | Type Property | Purpose                                                |
|-----------------|---------------|--------------------------------------------------------|
| ^^aff4:SHAKE128 | aff4:hash     | Support SHAKE128 message digest - Defined in FIPS 202  |
| ^^aff4:SHAKE256 | aff4:hash     | Support SHAKE256 message digest - Defined in FIPS 202  |
| ^^aff4:SHA3-256 | aff4:hash     | Support SHA3-256 message digest - Defined in FIPS 202  |
| ^^aff4:SHA3-384 | aff4:hash     | Support SHA3-384 message digest - Defined in FIPS 202  |
| ^^aff4:SHA3-512 | aff4:hash     | Support SHA3-512 message digest - Defined in FIPS 202  |

## 5. File name and path normalisation considerations

Due to differences in filesystem implementations, allowed/disallowed characters
in filenames in various Operating systems, and text encoding in TURTLE RDF to typically
being UTF-8, normalisation rules are required.

For example, it is generally recognised that UNIX system only disallow {NUL} (0x00)
and '/' (path separator) as filename characters, allowing the use of {TAB} (0x09),
{BS} (0x08). {CR} (0x0d), etc. Allowing these characters to be stored directly in a
string field in TURTLE RDF may cause issues with parsing RDF in future.

Therefore the following normalisation rules are present:

1. If the filename is valid UTF-8 and does not contains control characters (0x00 - 0x1f), the filename/pathname is stored as-is. aff4:fileNameRaw / aff4:originalPathNameRaw is not used.
2. If the filename is not valid UTF-8 or contains control characters (0x00 - 0x1f), then: 
   - a. The filename is base64 encoded in the stored in aff4:fileNameRaw / aff4:originalPathNameRaw respectively. 
   - b. The filename is treated as extended 8bit ASCII and has char/byte values 0x00 - 0x1f, 0x25 and 0x80 - 0xff replaced with '%{uppercase hex value of byte, padded with leading zero}', and stored in aff4:fileName / aff4:originalPathName, eg '{TAB}{CR}malware.exe{BS}{BS}{BS}txt' encodes as '%09%0Dmalware.exe%08%08%08txt'

It should be noted that when the filenamme / pathname is encoded no effort is required to deconflict with other files in the same folder.

If a filename/pathname is encoded as base64, the Application should take care in how the decoded base64 version is displayed to the user taking into account disallowed characters, reserved filenames, etc of the analysts system.

## 6. Data Storage Streams

Data streams found in current generation filesystems range from several bytes (eg NTFS ADS) to multi-terabyte (Virtual Machine Files). 
While the ImageStream and Map of the AFF4 Standard v1.0 are efficient for large files, when storing large numbers of small files, the overhead of storing data streams as ImageStreams or Maps becomes significant. 

For this reason, AFF4-L supplements these data storage streams with two efficient storage methods:
- In-metadata Storage Streams: store stream content within the AFF4 (turtle) metadata. This technique is analagous to resident data streams in NTFS. 
- ZipSegment Storage Streams: store stream content as a single standard ZIP file segment. 

Implementations which consume (ie read) AFF4-L containers MUST support all Data Storage Streams described in this section (Section 6).

Implementations which produce (ie write) AFF4-L containers MUST support at least one of the Data Storage Streams described in this section (Section 6).

### 6.1 ZipSegment Storage Streams

ZipSegment Storage Streams are an efficient method of storing data streams of small files in the range of kilobytes to megabytes in size, and in creating simple AFF4-L implementations. 

File Images may use ZipSegment Storage Streams to store their primary storage stream by adding the type aff4:ZipSegment to the RDF type of the instance representing the file. 

```
<aff4://28617d0f-2ca1-47c0-82cb-4a8374029805>
        a                        aff4:FileImage , aff4:Image , aff4:ZipSegment ;
        aff4:hash                "47e2194cbeb3b83eef3c09d0cab3c4c1bab08736"^^aff4:SHA1 , "f7c9a3c77792c010880c571bd2e7f7e7"^^aff4:MD5 ;
        aff4:lastWritten         "2020-09-06T23:36:22Z"^^xsd:dateTime ;
        aff4:size                "23"^^xsd:long .
```

The actual content of the stream is stored using the instance's ARN as the name of the Zip Segment in the root of the AFF4 container. 

```
> unzip -t example.aff4
Archive:  MaxOS-xattr.23b.aff4
aff4://28617d0f-2ca1-47c0-82cb-4a8374029805
    testing: information.turtle       OK
    testing: aff4://28617d0f-2ca1-47c0-82cb-4a8374029805   OK
    testing: container.description    OK
    testing: version.txt              OK
```

Compliant implementations MUST only use Stored (0) or Deflate (8) compression methods when storing the stream. 

In choosing to employ ZIP based storage streams, implementers should be aware that ZIP Segments are NOT efficiently seekable for large files when compressed.  

Compliant implementations SHOULD NOT use ZipSegment Storage Streams for storing streams larger than 1GiB in size.

Implementations which produce AFF4-L containers MUST store linear hashes of the stream using the aff4:hash property on the ZipSegment Storage Stream.

### 6.2 In-metadata Storage Streams

In-metadata Storage Streams are an efficient method of storing data streams of bytes to tens of bytes in size (commonly NTFS ADS and APFS xattr's are of this file size).

Compliant implementations MUST NOT use In-metadata Storage Streams for storing streams larger than 1Kb in size.

Compliant implementations MAY omit calculation and storage of hashes for In-metadata Storage Streams, 
relying on the Metadata Integrity Hash for integrity. 

```
<aff4://2585b4c0-340c-45e7-ad9e-0f688da77ef8>
        a aff4:FileImage , aff4:Image , aff4:ContiguousImage ;
        ...
        aff4l:alternateDataStream  <aff4://bfcecaf5-7272-4476-8f57-e72017271b1c> .
        
<aff4://bfcecaf5-7272-4476-8f57-e72017271b1c>
        a            aff4:FileSubStream, aff4:Image ;
        aff4:size    "11"^^xsd:long ;
        aff4:target  <aff4://2585b4c0-340c-45e7-ad9e-0f688da77ef8> ;
        aff4:name   "helloADS" ;
        aff4l:dataStream  "IkhlbGxvIEFEUyI="^^xsd:base64Binary .
```

```
<aff4://2585b4c0-340c-45e7-ad9e-0f688da77ef8>
        a aff4:FileImage , aff4:Image , aff4:ContiguousImage ;
        aff4:size    "11"^^xsd:long ;
        ...
        aff4l:dataStream  "IkhlbGxvIEFEUyI="^^xsd:base64Binary .
```

### 6.3 Map Storage Streams

File Images may store their primary data stream in an AFF4 Standard v1 Map by adding the aff4:Map type to the FileImage instances's type. 

The aff4:dependentStream property links to the ImageStream which the Map employs for storage. 

By using Map's, implementations may choose to store the datastreams of multiple source files in a single Image Stream.

```
<aff4://31aa452b-bddf-4257-a998-5bb75a80f9b1>
        a                      aff4:Image , aff4:FileImage , aff4:Map , aff4:ContiguousImage ;
        aff4:dependentStream   <aff4://0947bfd0-1265-42d3-95b6-8d7bed108e9b> ;
        aff4:lastWritten       "2020-08-27T01:45:28Z"^^xsd:dateTime ;
        aff4:hash              "f384a16360a84dc83f3621d12d2d10cc859c7d242814c27d5e0dc343078b88b8abd1d4ddb983572436e32e4b18fbe263f6e6adc048e4f3ba930ec4d25a472ff9"^^aff4:blockMapHashSHA512 ;
        aff4:mapHash           "02c3558b67c5564b704dbf5df95b9e047553396b429cb9d2d82d5734a9c4e4440ee6deee01195096991a015d8b6569b13f575546fb60411bcdea2c3e3b9b2f43"^^aff4:SHA512 ;
        aff4:mapIdxHash        "6dbb15c6a9a4c5e9d893a7cfca636bf3080e6b1a5259744d26cb0f5aef3c094cda4caadd1853573a5d5b39113e7cd7e0208f571a80bd1f8caee9d68f78d99db9"^^aff4:SHA512 ;
        aff4:mapPointHash      "9580ca7260cbcf88d8b0ac8b47b8f6fe8a52b6c2af917b000b4f6ba6c23f9bd26cc91d15afcc35a70665bde22d88e9cd32f9a6414a0bf8a8d844d5c38b60b252"^^aff4:SHA512 ;
        aff4:originalFileName  "\GovDocs\000\000785.html" ;
        aff4:size              "34473"^^xsd:long .
```
A FileImage may choose to represent its associated datastream indirectly via the aff4l:dataStream property, per the below example.  
```
<aff4://2585b4c0-340c-45e7-ad9e-0f688da77ef8>
        a aff4:FileImage , aff4:Image , aff4:ContiguousImage ;
        aff4:lastWritten       "2020-08-27T01:45:28Z"^^xsd:dateTime ;
        aff4:originalFileName  "\GovDocs\000\000785.html" ;
        aff4:size              "34473"^^xsd:long .
        ...
        aff4l:dataStream  <aff4://31aa452b-bddf-4257-a998-5bb75a80f9b1> .
        
<aff4://31aa452b-bddf-4257-a998-5bb75a80f9b1>
        a                      aff4:Map ;
        aff4:dependentStream   <aff4://0947bfd0-1265-42d3-95b6-8d7bed108e9b> ;
        aff4:hash              "f384a16360a84dc83f3621d12d2d10cc859c7d242814c27d5e0dc343078b88b8abd1d4ddb983572436e32e4b18fbe263f6e6adc048e4f3ba930ec4d25a472ff9"^^aff4:blockMapHashSHA512 ;
        aff4:mapHash           "02c3558b67c5564b704dbf5df95b9e047553396b429cb9d2d82d5734a9c4e4440ee6deee01195096991a015d8b6569b13f575546fb60411bcdea2c3e3b9b2f43"^^aff4:SHA512 ;
        aff4:mapIdxHash        "6dbb15c6a9a4c5e9d893a7cfca636bf3080e6b1a5259744d26cb0f5aef3c094cda4caadd1853573a5d5b39113e7cd7e0208f571a80bd1f8caee9d68f78d99db9"^^aff4:SHA512 ;
        aff4:mapPointHash      "9580ca7260cbcf88d8b0ac8b47b8f6fe8a52b6c2af917b000b4f6ba6c23f9bd26cc91d15afcc35a70665bde22d88e9cd32f9a6414a0bf8a8d844d5c38b60b252"^^aff4:SHA512 ;
        aff4:size              "34473"^^xsd:long .
```

#### 6.3.1 Hashes of Maps

Compliant writer implementations MUST calculate and store Block Map Hashes for each map, using either:
- the aff4:blockMapHash property on the Map, or
- the aff4:hash property on the Map, along with the datatype-scoped hash construction and algorithm (recommended). 

Two examples follow:
```   
<aff4://31aa452b-bddf-4257-a998-5bb75a80f9b1>
        a                      aff4:Map ;
        aff4:hash              "f384a16360a84dc83f3621d12d2d10cc859c7d242814c27d5e0dc343078b88b8abd1d4ddb983572436e32e4b18fbe263f6e6adc048e4f3ba930ec4d25a472ff9"^^aff4:blockMapHashSHA512 ;
        ...
        aff4:size              "34473"^^xsd:long .
```

```   
<aff4://31aa452b-bddf-4257-a998-5bb75a80f9b1>
        a                      aff4:Map ;
        aff4:blockMapHash      "81881586142185db75077628a77c65a3007f0802885ce135e77f68e40a4cb5588205b939d6862e9a3a82cdb29234db995c936f5d6d5871a39c5746e082927a35"^^aff4:SHA512 ;
        ...
        aff4:size              "34473"^^xsd:long .
```

Compliant reader implementations MUST support reading the above two synonyms, and be capable of verifying the block map hashes of all Maps and dependent ImageStreams in an image.  

### 6.4 ImageStream Storage Streams

FileImage instances may additionally have type aff4:ImageStream to store their primary stream as an ImageStream. 

## 7. Container naming convention

AFF4-L implementations may choose to signal that an image is conformant to the AFF4-L Standard v1.0 by using the ".aff4l" file extension. This is purely a hint.

## 8. Multi-part containers naming convention

AFF4 Standard v1.0 provides for multi-segment containers, stored across multiple ZIP files (see Section 7). There exists the need to quickly identify a set of AFF4 files which comprise such a container, based solely on file name. 

Implementations compliant to AFF4-L Standard v1.0 may signal their membership in such a single container by the following naming scheme. 
```
filename ( ".aff4" | ".aff4l" ) [ "." ordinal ] 
```
The filename may be any filename, but must be constant for the entire container (set of files). The ordinal is a decimal integer, starting at 1 and incrementing by 1 for each file. 

| Filename   | Description            |
|------------|------------------------|
| foo.aff4   | First written segment  |
| foo.aff4.1 | Second written segment |
| foo.aff4.2 | Third written segment  |
| ..         | ..                     |
| foo.aff4.9 | 9th written segment    |
| foo.aff4.10 | 10th written segment   |
| foo.aff4.11 | 11th written segment   |
| ..         | ..                     |
| foo.aff4.n | nth written segment    |

This naming scheme is purely a hint. Implementations must conform to AFF4 Standard v1.0. 

## 9. Metadata storage extensibility

The AFF4 Standard v1.0 provides for all container metadata to be stored in the file "information.turtle". For the purposes of this standard we refer to this as the Primary Information Store. 

In scaling AFF4-L to storing millions of file images, it was observed that the time taken to parse the RDF contents of the information.turtle file was non-trivial. A need exists for storing container metadata in locations other than the information.turtle file ("secondary information stores"). 

Implementations may supplement the information stored in the information.turtle file by using the aff4l:imports property. 
With reference to the example below, the aff4l:imports statement refers to the resource <aff4://7cbb47d0-b04c-42bc-8c04-87b7782739ad/information.turtle.2>. 
Noting that the Volume's ARN is aff4://7cbb47d0-b04c-42bc-8c04-87b7782739ad, under the ARN to ZIP mapping rules, this would refer to a file named "information.turtle.2", which resides in the same AFF4 container. 
We such referred to files "Secondary Information Stores".

Implementations MUST treat the triples in the "information.turtle" and any triples stored in files referred to by the aff4l:imports property as all belonging to the same RDF graph. 

```
@prefix :      <aff4://7cbb47d0-b04c-42bc-8c04-87b7782739ad> .
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix aff4:  <http://aff4.org/Schema#> .
@prefix aff4l: <http://aff4.org/Schema/2022/#> .

:       a                  aff4:ZipVolume ;
        aff4l:imports     <aff4://7cbb47d0-b04c-42bc-8c04-87b7782739ad/information.turtle.2> ;
        aff4:stored        "Base-Linear_1.aff4" .
        
```

## 9a. Metadata storage scaling.

As foreshadowed in [1], parsing the primary information store when it contains information describing millions of files may consume significant CPU and RAM resources leading to untenable delays in opening AFF4-L images.

To avoid such delays, implementations MAY store a functionally equivalent version of the Primary Information Store using AFF4/HDT Serialisation as the file “information.hdt” alongside the Primary Information Store. This is referred to as the Accelerated Primary Information Store.

Should implementations use Secondary Information Stores per section 9 of this document and provide an Accelerated Primary Information Store, the latter must contain the totality of all information contained in the Primary Information Store and Secondary information Stores (ie. there should be no other AFF4/HDT information stores than the “Accelerated Primary Information Store” in an AFF4/L image).

Read implementations of the AFF4-L standard MAY support reading metadata from the Accelerated Primary Information Store instead of the totality of the Primary Information Store and Secondary information Stores.

## 9a.1 The AFF4/HDT RDF Serialisation

The AFF4/HDT Serialisation is a slight variation on the HDT Binary Format [6], modified to support UTF-8 strings (the original format relies on strings ending in a NUL to determine string length).

Implementations supporting the AFF4/HDT RDF Serialisation MUST limit their usage of HDT features to the following:

* Triples: must be stored using BitmapTriples <http://purl.org/HDT/hdt#triplesBitmap>
* Dictionary: must be stored using prefix and suffix coded Four Section Dictionary <http://purl.org/HDT/hdt#dictionaryFourPsfc>  (defined in the hdt-java implementation [8])
* DictionarySection: must be stored using PlainFrontCodingUtf8 (defined below).

The newly defined PlainFrontCodingUtf8 = 7 DictionarySection is for the length-prepended string encoding needed for tolerating Unicode NUL characters and UTF-8 in general.

With reference to the data definition DictionarySectionPlainFrontCoding at [6] DictionarySectionPlainFrontCodingUtf8 extends the definition of to employ length-prefixed strings rather than null-terminated strings, per the following:

```
<Buffer of Blocks>
 <VByte size of following string in bytes>
 <First String of Block>
 <VByte Number of Common prefix characters> <VByte size of following string in bytes> <Suffix String>
 … as many times as necessary to fit “BlockSize” strings.
<CRC32 of Buffer of Blocks>
```

Where there is ambiguity or difference between the above described AFF4/HDT Serialisation and the Reference Images, the reference images take precedence.

## 10. Metadata Integrity Hash

The hashing approach of AFF4 focussed on assuring the integrity of the Map and ImageStream constructs, with the top-level hashes of those being stored in the information.turtle file.

With AFF4-L much of the acquired data is stored in the metadata layer, when compared with AFF4 physical images. There exists a need for assuring the integrity of the metadata layer. 

### 10.1 Container Hash

Implementations MUST store the hash (or hashes) of the "information.turtle" file segment in a file alongside it, named "information.turtle.hashes". The content of that file is the hash, or hashes of the information.turtle file, formatted using the turtle RDF datatype syntax. For example:

information.turtle.hashes:
```
@prefix :      <aff4://7cbb47d0-b04c-42bc-8c04-87b7782739ad> .
@prefix aff4:  <http://aff4.org/Schema#> .

:/information.turtle 
      aff4:hash      "440ee6deee01195096991a015d8b6569b13f575546fb60411bcdea2c3e3b9b2f"^^aff4:SHA256 ;
      aff4:hash      "6dbb15c6a9a4c5e9d893a7cfca636bf3080e6b1a5259744d26cb0f5aef3c094cda4caadd1853573a5d5b39113e7cd7e0208f571a80bd1f8caee9d68f78d99db9"^^aff4:SHA512 .
```

The hash (or hashes) used MUST be SHA256, SHA512 or stronger hash supported by this standard.

### 10.2 Container signing

Implementations MAY choose to store an X509 signature of the "information.turtle" file segment in a file alongside it, named "information.turtle.sig". 
The signature MUST use PEM encoding. When signing, the full public certificate chain including all intermediate and root CA certificates MUST be 
stored alongside the signature, in PEM encoding, with the name "information.turtle.cer".

```
/information.turtle
/information.turtle.sig
/information.turtle.cer
```

Should an implementation wish to support signing the information.turtle file with multiple keys, a compliant 
implementation MUST construct the signature and certificate names as follows:

```
SIGNATURE = "/information.turtle." + extensible-part + ".sig"
CERTIFICATE = "/information.turtle." + extensible-part + ".cer"
```

The extensible-part is defined by the implementer and MUST be common for a related signature and certificate chain.
In addition, the extensible part must conform to valid UTF-8.

An example follows:
```
/information.turtle
/information.turtle.name1.sig
/information.turtle.name1.cer
/information.turtle.name2.sig
/information.turtle.name2.cer
```
Where an Accelerated Primary Information Store is employed (ie and “information.hdt” exists alongside an “information.turtle”), implementations may choose to sign the “information.hdt” following the aforementioned pattern.

### 10.3 Hashing of Secondary Information Stores

Where an implementation employs Secondary Information Stores, the hashes of those stores must be stored in the Primary Information Store by using the resource name of the Secondary Information Store, along with the aff4:hash property, per the following example:

```
@prefix :      <aff4://7cbb47d0-b04c-42bc-8c04-87b7782739ad> .
@prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix aff4:  <http://aff4.org/Schema#> .
@prefix aff4l: <http://aff4.org/Schema/2022/#> .

:       a                  aff4:ZipVolume ;
        aff4l:imports     <aff4://7cbb47d0-b04c-42bc-8c04-87b7782739ad/information.turtle.2> ;
        aff4:stored        "Base-Linear_1.aff4" .

:/information.turtle.2 
        aff4:hash       "1af482ce0f7237d2cd8bdda38d6550e4a42878e8d8a1854e85dd8beb5caa3567f6c0b96f76f6a455a4d021e506e29fbce358f0170edcdcd03e3b15b4a4b3953f"^^aff4:SHA512 . 
```
The hash used MUST be SHA256, SHA512 or stronger hash supported by this standard.

## 11. References

[1] Schatz (2019), "AFF4-L: A scalable open logical evidence container", https://dfrws.org/presentation/aff4-l-a-scalable-open-logical-evidence-container/

[2] Cohen, Garfinkel, Schatz (2009) “Extending the Advanced Forensic Format to Accommodate Multiple Data Sources, Logical Evidence, Arbitrary Information and Forensic Workflow”
http://dfrws.org/conferences/dfrws-usa-2009/sessions/extending-advanced-forensic-format-accommodate-multiple-data

[3] Cohen & Schatz (2010) “Hash based disk imaging using AFF4”
http://dfrws.org/conferences/dfrws-usa-2010/sessions/hash-based-disk-imaging-using-aff4

[4] Schatz (2015) “Wirespeed: Extending The AFF4 Container Format For Scalable Acquisition And Live Analysis”
http://dfrws.org/conferences/dfrws-usa-2015/sessions/wirespeed-extending-aff4-container-format-scalable-acquisition

[5] pyAFF4 https://github.com/aff4/pyaff4

[6] HDT Binary Format DRAFT VERSION 03/07/2015 https://www.rdfhdt.org/hdt-binary-format/ 

[7] hdt-rs https://github.com/KonradHoeffner/hdt

[8] hdt-java/PSFCFourSectionDictionary.java https://github.com/rdfhdt/hdt-java/blob/master/hdt-java-core/src/main/java/org/rdfhdt/hdt/dictionary/impl/PSFCFourSectionDictionary.java

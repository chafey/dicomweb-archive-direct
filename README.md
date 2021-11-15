# dicomweb-archive-direct
Specification for a DICOMweb based archive format

# Introduction

DICOM currently has a single on disk file format defined in P10 of the standard.  P10 was created to enable interoperability at the physical media level - specifically CD, DVD and USB sticks.  It is often used as an archive format as it is the only on standard on disk format but suffers from a number of issues when used this way (see below).  With the introduction of
DICOMweb, it makes sense to explore a modern archive format that addresses the issues with DICOM P10 and is more web and cloud friendly.

# Challenges with DICOM P10 as an archive format

* Data Denormalization - patient, study and series information is stored in each SOP Instance (DICOM P10 file) leading to duplication of data and possibilty of data inconsistences
* Inefficient updates - Since DICOM P10 stores the metadata in the front of the file before the pixel data, any updates (e.g. patient name change) require the entire study to be rewritten which requries significant resources (I/O in particular)
* Inefficient image frame access - Since image frames are stored at the end of the file and can possibly be fragmented, accessing it requries parsing the entire data set and reassembling the frame before returning it to a client
* Inconsistent image frame encoding - A given SOP Instance may contain a single frame or multiple frames.  For multi-frame instances, it may or may not include a basic offset table
* Inconsistent parse performance.  A DICOM P10 file may be encoded in a variety of formats, some of which are much slower to parse than others (e.g. undefined length sequences)
* Inconsistent type information.  A DICOM P10 file may be encoded in explicit or implicit formats.  Implicit formats do no include the VR leading to ambiguities for certain attributes such
as private tags and standard tags that can be more than one VR (e.g. LUT Data)

# Proposal

Compliant archives will provide direct access via HTTP GET or the file system to the following DICOMweb resources:

* Study Metadata
  * DICOM JSON encoding with VRs included (Explicit format)
  * gzipped
  * Denormalized data cannot have any inconsistencies (all patient names must be the same)
* Image Frames - in any supported transfer sytnax, wrapped in multi-part mime header
* Bulk Data - wrapped in multi-part mime header

The archive will provide direct access to the above resources - either via HTTP GET or by reading the file system directly.  The direct archive mechanism will not support any
on the fly modification of the data (e.g. no transcoding to another transfer syntax or encoding)

An entire study can be archived as a single file by storing all of the above resources in a single ZIP file.  The resources will use file paths
that match the WADO-RS URIs

NOTE: Semantically equivalent DICOM P10 can be generated from the above resources

# FAQ

### Q: Why don't you also store series and/or sop instance metadata?
### A: It is unnecessary and could lead to data inconsistencies.  These objects can easily be created from the study metadata, but are unnecessary as part of the archive format

### Q: Why don't you support QIDO-RS responses such as Study Series and Series Instances?
### A: It is unnecessary and could lead to data inconsistencies.  These objects can easily be created from the study metadata, but are unnecessary as part of the archive format

### Q: Why use ZIP as your study archive format?
### A: For consistency with the existing ZIP funcionality in DICOMweb

### Q: Why not create a new study metadata format that is actually denormalized
### A: To minimize the size of this specific standard proposal.  If a denormalized study metadata object is standardized in the future, it should be possible to use it as part of this standard

### Q: What does a semantically equivalent DICOM P10 file mean?
### A: Parsing the generated DICOM P10 file will return the exact same values, but the actual bits may be different due to variations in DICOM P10 encoding formats (e.g. undefined length sequences).  The details of the encoding mechanism are not preserved so bitwise equivalence is not possible
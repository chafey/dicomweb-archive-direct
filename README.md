# dicomweb-archive-direct
Specification for a DICOMweb based archive format

# Introduction

DICOM currently has a single on disk file format defined in P10 of the standard.  P10 was created to enable interoperability at the physical media level - specifically CD, DVD and USB sticks.  It is often used as an archive format as it is the only on standard on disk format but suffers from a number of issues when used this way (see below).  With the introduction of
DICOMweb, it makes sense to explore a modern archive format that is addresses the issues with P10 and is more web and cloud friendly.

# Challenges with DICOM P10 as an archive format

* Data Denormalization - patient, study and series information is stored in each SOP Instance (DICOM P10 file) leading to duplication of data and possibilty of data inconsistences
* Inefficient updates - Since DICOM P10 stores the metadata in the front of the file before the pixel data, any updates (e.g. patient name change) require the entire study to be rewritten which requries significant resources (I/O in particular)
* Inefficient image frame access - Since image frames are stored at the end of the file and can possibly be fragmented, accessing it requries parsing the entire data set and reassembling the frame before returning it to a client
* Inconsistent image frame encoding - A given SOP Instance may contain a single frame or multiple frames.  For multi-frame instances, it may or may not include a basic offset table
* Inconsistent parse performance.  A DICOM P10 file may be encoded in a variety of formats, some of which are much slower to parse than others (e.g. undefined length sequences)
* Inconsistent type information.  A DICOM P10 file may be encoded in explicit or implicit formats.  Implicit formats do no include the VR leading to ambiguities for certain attributes such
as private tags and standard tags that can be more than one VR (e.g. LUT Data)

# Proposal

Compliant archives will provide direct HTTP GET access to the following DICOMweb resources:

* Study Metadata - in DICOM JSON encoding, gzipped
* Image Frames - in any supported transfer sytnax, wrapped in multi-part mime header
* Bulk Data - wrapped in multi-part mime header

The above resources will be returned without any modification (e.g. no transcoding to another transfer syntax or encoding format)

An entire study can be archived as a single file by storing all of the above resources in a single ZIP file.  The resources will use file paths
that match the WADO-RS URIs

# CDR File Format Specification

CDR is a File format used by CorelDRAW vector graphics editor since 1989 until today. Authors did not release any public description of the format. This is the "unofficial" specification.

Several groups have claimed, that they have parsed the CDR format (libcdr, sK1 Project). However, instead of publishing a clear description, they force users into utilizing their software tools, which they created. This is the first human-readable description of the CDR format.

There are two categories of CDR files:
- Start with `WL`: Old Format - used before 1992
- Start with `RIFF`: RIFF Format - used after 1992

# Old Format
- starts with ASCII bytes `WL`

# RIFF Format

The file structure corresponds to the RIFF format. Some "LIST" chunks contain data, which can not be interpreted as subchunks.

The file is a sequence of bytes. Following data types are used in this document:
- `asci4` - four bytes, interpreted as a four-byte ASCII string
- `uint4` - four bytes, interpreted as a 32-bit unsigned integer in Little Endian

## File Structure

    Size  Type    Value
    4     asci4   "RIFF"
    4     uint4   The length of the following data (file size - 8)
    4     asci4   "CDR" + a byte, identifying the format version
    *     CHUNKS  The sequence of chunks

A chunk is a sequence of bytes in the following format

    Size  Type    Value
    4     asci4   Identifier of the chunk
    4     uint4   N : the size of the chunk content
    N     bytes   The content of the chunk

When N is odd, there is a one-byte padding before the next chunk.

When the chunk identifier is "LIST", the content starts with asci4, which is called a **list-type**.
    
The CDR file contains six chunks (starting at byte 12): 

    ID     Size   list-type
    vrsn   2      
    DISP   *      
    LIST   24     INFO
    LIST   52     cmpr
    LIST   *      cmpr
    sumi   60

The main data is in the fifth chunk. The content of important chunk types is described below.

## DISP
Contains a raster image: a thumbnail of the CDR file.

    Size  Type    Value
    8     bytes   unknown
    4     uint4   W: width
    4     uint4   H: height
    28    bytes   unknown
    1024  bytes   Palette for the image (256 samples * 4 bytes per sample)
    W*H   bytes   Indices to the palette for each pixel

## LIST X  -  X is not "cmpr" or "stlt"
Contains subchunks.

    Size  Type    Value
    4     asci4   list-type
    *     CHUNKS  The sequence of chunks
    
## LIST cmpr
Contains compressed subchunks.

    Size  Type    Value
    4     asci4   "cmpr"
    4     uint4   C1 - the size of Part1 compressed
    4     uint4   U1 - the size of Part1 uncompressed
    4     uint4   C2 - the size of Part2 compressed
    4     uint4   U2 - the size of Part2 uncompressed
    
    4     asci4   "CPng"
    4     bytes   0x01, 0x00, 0x04, 0x00
    C1-8  bytes   Part1 compressed with ZLIB
    4     asci4   "CPng"
    4     bytes   0x01, 0x00, 0x04, 0x00
    C2-8  bytes   Part1 compressed with ZLIB

This chunk contains two (decompressed) byte sequences: Part1 and Part2. Part2 is a list of uint4 numbers.

Part1 is `CHUNKS`: a sequence of chunks with a small difference: the Size of each chunk is not the actual size, but the index into Part2, where the true Size is stored.

## LIST stlt
Some weird structure.

    Size  Type    Value
    4     asci4   "stlt"
    *     bytes   unknown


    
    

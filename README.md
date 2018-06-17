# CDR File Format Specification

CDR is a File format used by CorelDRAW vector graphics editor since 1989 until today. Authors did not release any public description of the format. This is the "unofficial" specification.

Several groups have claimed, that they have parsed the CDR format (libcdr, sK1 Project). However, instead of publishing a clear description, they force users into utilizing their software tools, which they created. This is the first human-readable description of the CDR format.

There are two categories of CDR files:
- Start with `WL`: Old Format - used before 1992
- Start with `RIFF`: RIFF Format - used after 1992

## Old Format
- starts with ASCII bytes `WL`

## RIFF Format

The file structure corresponds to the RIFF format. Some "LIST" chunks contain data, which can not be interpreted as subchunks.

The file is a sequence of bytes. Following data types are used in this document:
- `asci4` - four bytes, interpreted as a four-byte ASCII string
- `uint4` - four bytes, interpreted as a 32-bit unsigned integer in Little Endian

### File Structure

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

When the size of a chunk is odd, there is a one-byte padding before the next chunk.

When the chunk identifier is "LIST", the content starts with asci4, which is called a **list-type**.
    
The CDR file contains six chunks (starting at byte 12): 

    ID     Size   list-type
    vrsn   2      
    DISP   *      
    LIST   24     INFO
    LIST   52     cmpr
    LIST   *      cmpr
    sumi   60
 
 
    
    
    

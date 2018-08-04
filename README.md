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

Each CRD file has a version **V** associated with it. The structure of some parts of the file may depend on V.

The file is a sequence of bytes. Following data types are used in this document:
- `asci4` - four bytes, interpreted as a four-byte ASCII string
- `uint2` - two bytes, interpreted as a 16-bit unsigned integer in Little Endian
- `uint4` - four bytes, interpreted as a 32-bit unsigned integer in Little Endian
- `sint2` - two bytes, interpreted as a 16-bit signed integer in Little Endian
- `sint4` - four bytes, interpreted as a 32-bit signed integer in Little Endian
- `doubl` - 64-bit floating point number in Little Endian
- `coord` - if V<600, the value is sint2 / 1000,  otherwise sint4 / 254 000
- `string` - the sequence of non-zero bytes, ended with a zero byte, represents an ASCII text

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
    
The CDR file usually consists of six chunks (starting at byte 12): 

    ID     Size   list-type
    vrsn   2      
    DISP   *      
    LIST   24     INFO
    LIST   52     cmpr
    LIST   *      cmpr
    sumi   60

The main data is in the second "LIST cmpr" chunk. The content of important chunk types is described below.

## vrsn 
Contains a **version** (below as **V**) of the file

    Size  Type    Value
    2     uint2   Version of the file
    
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
    C2-8  bytes   Part2 compressed with ZLIB

This chunk contains two (decompressed) byte sequences: Part1 and Part2. Part2 is a list of uint4 numbers.

Part1 is `CHUNKS`: a sequence of chunks with a small difference: the Size of each chunk is not the actual size, but the index into Part2, where the true Size is stored.

## LIST stlt
**ST**y**L**es of **T**ext

    Size  Type    Value
    4     asci4   "stlt"
    4     uint4   Number of Records (NOR)
    *     Map12   Fill IDs
    *     Map12   Outline IDs
    
    4     uint4   Number of Fonts (NOF)
    // repeated NOF times
    4     uint4   Font Style ID
    *     bytes   unknown.  Size: V<1000 ? 12 : 20
    4     uint4   Font ID
    4     uint4   Font Encoding
    8     bytes   unknown
    *     coord   Font Size
    *     bytes   unknown.  Size: V<1000 ? 12 : 20
    //  end repeat
    *     Map12   Align IDs
    4     uint4   Number of Intervals (NOI)
    52*NOI        unknown
    4     uint4   Number of Set5 (NOS)
    152*NOS       unknown
    4     uint4   Number of Tabs (NOT)
    784*NOT       unknown
    
    4     uint4   Number of Bullets  (NOB)
    //  repeat NOB times
    40    byets   unknown
    4     uint4   unknown  (only when V>1300)
    //  if  V>=1300
    4     uint4   X       
    68    bytes   unknown  (if X not 0)
    12    bytes   unknown  (if X ==  0)
    //  else 
    20    bytes   unknown
    8     bytes   unknown  (if V>=1000)
    4     uint4   X
    8     bytes   unknown  (if X not 0)
    8     bytes   unknown
    //  end if
    //  end repeat
    
    4     uint4   Number of Indents (NOI)
    //  repeat NOI times
    4     uint4   Indent ID
    12    bytes   unknown
    *     coord   right indent
    *     coord   first indent
    *     coord   left indent
    //  end repeat
    
    4     uint4   Number of Hypens (NOH)
    //  repeat NOH times
    32    bytes   unknown
    4     bytes   unknown (if V>=1300)
    //  end repeat
    
    4     uint4   Number of Dropcaps (NOD)
    28*NOD        unknown
    
    4     uint4   Number of Set11  (NOS, if V>800)
    12*NOS        unknown  (if V>800)
    
    //  repeat NOR times 
    4     uint4   NUM
    4     uint4   Style ID
    4     uint4   Parent ID
    4     uint4   NL
    NL    bytes   unknown 
    NL    bytes   unknown (if V>=1200)
    4     uint4   Fill ID
    4     uint4   Outline ID
    // if NUM > 1
    4     uint4   Font Rec ID
    4     uint4   Align ID
    4     uint4   Interval ID
    4     uint4   Set5 ID
    4     uint4   Set11 ID  (if V>800)
    // end if
    // if NUM > 2
    4     uint4   Tab ID
    4     uint4   Bullet ID
    4     uint4   Indent ID
    4     uint4   Hyphen ID
    4     uint4   Drop Cap ID
    // end if
    
Map12 has a following structure:

    4     uint4   Number of Values (NOV)
    // repeated NOV times:
    4     uint4   ID
    4     uint4   unknown
    4     uint4   Value
    48    bytes   unknown data (only for Fill IDs, when V>=1300)
    
## font
Data of a single font

    4     uint2   Font ID
    4     uint2   Encoding ID
    14    bytes   unknown
    *     string  Font name
    
## mcfg
Configuration

    *     bytes   unknown. Size (if V>=1300 : 12,  else if V>=900 : 4,  else if  700>V>=600 : 28)
    *     coord   Default page width
    *     coord   Default page height
    
## loda
Shape data  (geometry + fills + outlines)
    

    
    

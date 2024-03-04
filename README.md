# Hacking: The Art of Exploitation

_Code (and text) from [**Hacking book by Jon Erickson**](https://nostarch.com/hacking2)._

<div align="center" width="100%">
<img src="https://github.com/damianiRiccardo90/Hacking/blob/master/Hacking_Forepage.png?raw=true" alt="Hacking Forepage">
</div>

---

- [Chapter 0x100] [**Introduction**](C1-Introduction/Intro.md)  
- [Chapter 0x200] [**Programming**](C2-Programming/Intro.md)  
  - [Section 0x210] [**What is Programming?**](C2-Programming/C2-S1-What_Is_Programming/Intro.md)  
  - [Section 0x220] [**Pseudo-code**](C2-Programming/C2-S2-Pseudo-code/Intro.md)  
  - [Section 0x230] [**Control Structures**](C2-Programming/C2-S3-Control_Structures/Intro.md)  
    - [Paragraph 0x231] [**If-Then-Else**](C2-Programming/C2-S3-Control_Structures/If-Then-Else.md)  
    - [Paragraph 0x232] [**While/Until Loops**](C2-Programming/C2-S3-Control_Structures/While-Until_Loops.md)  
    - [Paragraph 0x233] [**For Loops**](C2-Programming/C2-S3-Control_Structures/For_Loops.md)  
  - [Section 0x240] [**More Fundamental Programming Concepts**](C2-Programming/C2-S4-More_Fundamental_Programming_Concepts/Intro.md)  
    - [Paragraph 0x241] [**Variables**](C2-Programming/C2-S4-More_Fundamental_Programming_Concepts/Variables.md)  
    - [Paragraph 0x242] [**Arithmetic Operators**](C2-Programming/C2-S4-More_Fundamental_Programming_Concepts/Arithmetic_Operators.md)  
    - [Paragraph 0x243] [**Comparison Operators**](C2-Programming/C2-S4-More_Fundamental_Programming_Concepts/Comparison_Operators.md)  
    - [Paragraph 0x244] [**Functions**](C2-Programming/C2-S4-More_Fundamental_Programming_Concepts/Functions.md)  
  - [Section 0x250] [**Getting Your Hands Dirty**](C2-Programming/C2-S5-Getting_Your_Hands_Dirty/Intro.md)  
    - [Paragraph 0x251] [**The Bigger Picture**](C2-Programming/C2-S5-Getting_Your_Hands_Dirty/The_Bigger_Picture.md)  
    - [Paragraph 0x252] [**The x86 Processor**](C2-Programming/C2-S5-Getting_Your_Hands_Dirty/The_x86_Processor.md)  
    - [Paragraph 0x253] [**Assembly Language**](C2-Programming/C2-S5-Getting_Your_Hands_Dirty/Assembly_Language.md)  
  - [Section 0x260] [**Back to Basics**](C2-Programming/C2-S6-Back_To_Basics/Intro.md)  
    - [Paragraph 0x261] [**Strings**](C2-Programming/C2-S6-Back_To_Basics/Strings.md)  
    - [Paragraph 0x262] [**Signed, Unsigned, Long, and Short**](C2-Programming/C2-S6-Back_To_Basics/Signed_Unsigned_Long_And_Short.md)  
    - [Paragraph 0x263] [**Pointers**](C2-Programming/C2-S6-Back_To_Basics/Pointers.md)  
    - [Paragraph 0x264] [**Format Strings**](C2-Programming/C2-S6-Back_To_Basics/Format_Strings.md)  
    - [Paragraph 0x265] [**Typecasting**](C2-Programming/C2-S6-Back_To_Basics/Typecasting.md)  
    - [Paragraph 0x266] [**Command-Line Arguments**](C2-Programming/C2-S6-Back_To_Basics/Command-Line_Arguments.md)  
    - [Paragraph 0x267] [**Variable Scoping**](C2-Programming/C2-S6-Back_To_Basics/Variable_Scoping.md)  
  - [Section 0x270] [**Memory Segmentation**](C2-Programming/C2-S7-Memory_Segmentation/Intro.md)  
    - [Paragraph 0x271] [**Memory Segments in C**](C2-Programming/C2-S7-Memory_Segmentation/Memory_Segments_In_C.md)  
    - [Paragraph 0x272] [**Using the Heap**](C2-Programming/C2-S7-Memory_Segmentation/Using_The_Heap.md)  
    - [Paragraph 0x273] [**Error-Checked malloc()**](C2-Programming/C2-S7-Memory_Segmentation/Error_Checked_Malloc.md)  
  - [Section 0x280] [**Building on Basics**](C2-Programming/C2-S8-Building_On_Basics/Intro.md)  
    - [Paragraph 0x281] [**File Access**](C2-Programming/C2-S8-Building_On_Basics/File_Access.md)  
    - [Paragraph 0x282] [**File Permissions**](C2-Programming/C2-S8-Building_On_Basics/File_Permissions.md)  
    - [Paragraph 0x283] [**User IDs**](C2-Programming/C2-S8-Building_On_Basics/User_IDs.md)  
    - [Paragraph 0x284] [**Structs**](C2-Programming/C2-S8-Building_On_Basics/Structs.md)  
    - [Paragraph 0x285] [**Function Pointers**](C2-Programming/C2-S8-Building_On_Basics/Function_Pointers.md)  
    - [Paragraph 0x286] [**Pseudo-random Numbers**](C2-Programming/C2-S8-Building_On_Basics/Pseudo-random_Numbers.md)  
    - [Paragraph 0x287] [**A Game of Chance**](C2-Programming/C2-S8-Building_On_Basics/A_Game_Of_Chance.md)  
- [Chapter 0x300] [**Exploitation**](C3-Exploitation/Intro.md)  
  - [Section 0x310] [**Generalized Exploit Techniques**](C3-Exploitation/C3-S1-Generalized_Exploit_Techniques/Intro.md)  
  - [Section 0x320] [**Buffer Overflows**](C3-Exploitation/C3-S2-Buffer_Overflows/Intro.md) 
    - [Paragraph 0x321] [**Stack-Based Buffer Overflow Vulnerabilities**](C3-Exploitation/C3-S2-Buffer_Overflows/Stack-Based_Buffer_Overflow_Vulnerabilities.md)  
  - [Section 0x330] [**Experimenting with BASH**](C3-Exploitation/C3-S3-Experimenting_With_BASH/Intro.md)  
    - [Paragraph 0x331] [**Using the Environment**](C3-Exploitation/C3-S3-Experimenting_With_BASH/Using_The_Environment.md)  
  - [Section 0x340] [**Overflows in Other Segments**](C3-Exploitation/C3-S4-Overflows_In_Other_Segments/Intro.md)  
    - [Paragraph 0x341] [**A Basic Heap-Based Overflow**](C3-Exploitation/C3-S4-Overflows_In_Other_Segments/A_Basic_Heap_Based_Overflow.md)  
    - [Paragraph 0x342] [**Overflowing Function Pointers**](C3-Exploitation/C3-S4-Overflows_In_Other_Segments/Overflowing_Function_Pointers.md)  
  - [Section 0x350] [**Format Strings**](C3-Exploitation/C3-S5-Format_Strings/Intro.md)  
    - [Paragraph 0x351] [**Format Parameters**](C3-Exploitation/C3-S5-Format_Strings/Format_Parameters.md)  
    - [Paragraph 0x352] [**The Format String Vulnerability**](C3-Exploitation/C3-S5-Format_Strings/The_Format_String_Vulnerability.md)  
    - [Paragraph 0x353] [**Reading from Arbitrary Memory Addresses**](C3-Exploitation/C3-S5-Format_Strings/Reading_From_Arbitrary_Memory_Addresses.md)  
    - [Paragraph 0x354] [**Writing to Arbitrary Memory Addresses**](C3-Exploitation/C3-S5-Format_Strings/Writing_To_Arbitrary_Memory_Addresses.md)  
    - [Paragraph 0x355] [**Direct Parameter Access**](C3-Exploitation/C3-S5-Format_Strings/Direct_Parameter_Access.md)  
    - [Paragraph 0x356] [**Using Short Writes**](C3-Exploitation/C3-S5-Format_Strings/Using_Short_Writes.md)  
    - [Paragraph 0x357] [**Detours with .dtors**](C3-Exploitation/C3-S5-Format_Strings/Detours_With_Dtors.md)  
    - [Paragraph 0x358] [**Another notesearch Vulnerability**](C3-Exploitation/C3-S5-Format_Strings/Another_Notesearch_Vulnerability.md)  
    - [Paragraph 0x359] [**Overwriting the Global Offset Table**](C3-Exploitation/C3-S5-Format_Strings/Overwriting_The_Global_Offset_Table.md)  
- [Chapter 0x400] [**Networking**](C4-Networking/Intro.md)  
  - [Section 0x410] [**OSI Model**](C4-Networking/C4-S1-OSI_Model/Intro.md)  :godmode:
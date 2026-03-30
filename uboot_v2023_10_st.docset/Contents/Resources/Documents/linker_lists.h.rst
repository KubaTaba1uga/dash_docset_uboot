.. -*- coding: utf-8; mode: rst -*-

linker_lists.h
==============

.. parsed-literal::

    \/\* SPDX-License-Identifier\: GPL-2.0+ \*\/
    \/\*
     \* include\/linker\_lists.h
     \*
     \* Implementation of linker-generated arrays
     \*
     \* Copyright (C) 2012 Marek Vasut \<marex@denx.de\>
     \*\/

    \#ifndef \ :ref:`__LINKER_LISTS_H__ <--linker-lists-h-->`
    \#define \ :ref:`__LINKER_LISTS_H__ <--linker-lists-h-->`

    \#include \<linux\/compiler.h\>

    \/\*
     \* There is no use in including this from ASM files.
     \* So just don't define anything when included from ASM.
     \*\/

    \#if !defined(\_\_ASSEMBLY\_\_)

    \/\*\*
     \* llsym() - Access a linker-generated array entry
     \* @\_type\:      Data type of the entry
     \* @\_name\:      Name of the entry
     \* @\_list\:      name of the list. Should contain only characters allowed
     \*              in a C variable name!
     \*\/
    \#define llsym(\_type, \_name, \_list) \\
                    ((\_type \*)\&\_u\_boot\_list\_2\_\#\#\_list\#\#\_2\_\#\#\_name)

    \/\*\*
     \* ll\_entry\_declare() - Declare linker-generated array entry
     \* @\_type\:      Data type of the entry
     \* @\_name\:      Name of the entry
     \* @\_list\:      name of the list. Should contain only characters allowed
     \*              in a C variable name!
     \*
     \* This macro declares a variable that is placed into a linker-generated
     \* array. This is a basic building block for more advanced use of linker-
     \* generated arrays. The user is expected to build their own macro wrapper
     \* around this one.
     \*
     \* A variable declared using this macro must be compile-time initialized.
     \*
     \* Special precaution must be made when using this macro\:
     \*
     \* 1) The \_type must not contain the "static" keyword, otherwise the
     \*    entry is generated and can be iterated but is listed in the map
     \*    file and cannot be retrieved by name.
     \*
     \* 2) In case a section is declared that contains some array elements AND
     \*    a subsection of this section is declared and contains some elements,
     \*    it is imperative that the elements are of the same type.
     \*
     \* 3) In case an outer section is declared that contains some array elements
     \*    AND an inner subsection of this section is declared and contains some
     \*    elements, then when traversing the outer section, even the elements of
     \*    the inner sections are present in the array.
     \*
     \* Example\:
     \*
     \* \:\:
     \*
     \*   ll\_entry\_declare(struct my\_sub\_cmd, my\_sub\_cmd, cmd\_sub) = \{
     \*           .x = 3,
     \*           .y = 4,
     \*   \};
     \*\/
    \#define ll\_entry\_declare(\_type, \_name, \_list)                           \\
            \_type \_u\_boot\_list\_2\_\#\#\_list\#\#\_2\_\#\#\_name \_\_aligned(4)           \\
                            \_\_attribute\_\_((unused))                         \\
                            \_\_section("\_\_u\_boot\_list\_2\_"\#\_list"\_2\_"\#\_name)

    \/\*\*
     \* ll\_entry\_declare\_list() - Declare a list of link-generated array entries
     \* @\_type\:      Data type of each entry
     \* @\_name\:      Name of the entry
     \* @\_list\:      name of the list. Should contain only characters allowed
     \*              in a C variable name!
     \*
     \* This is like ll\_entry\_declare() but creates multiple entries. It should
     \* be assigned to an array.
     \*
     \* \:\:
     \*
     \*   ll\_entry\_declare\_list(struct my\_sub\_cmd, my\_sub\_cmd, cmd\_sub) = \{
     \*        \{ .x = 3, .y = 4 \},
     \*        \{ .x = 8, .y = 2 \},
     \*        \{ .x = 1, .y = 7 \}
     \*   \};
     \*\/
    \#define ll\_entry\_declare\_list(\_type, \_name, \_list)                      \\
            \_type \_u\_boot\_list\_2\_\#\#\_list\#\#\_2\_\#\#\_name[] \_\_aligned(4)         \\
                            \_\_attribute\_\_((unused))                         \\
                            \_\_section("\_\_u\_boot\_list\_2\_"\#\_list"\_2\_"\#\_name)

    \/\*
     \* We need a 0-byte-size type for iterator symbols, and the compiler
     \* does not allow defining objects of C type 'void'. Using an empty
     \* struct is allowed by the compiler, but causes gcc versions 4.4 and
     \* below to complain about aliasing. Therefore we use the next best
     \* thing\: zero-sized arrays, which are both 0-byte-size and exempt from
     \* aliasing warnings.
     \*\/

    \/\*\*
     \* ll\_entry\_start() - Point to first entry of linker-generated array
     \* @\_type\:      Data type of the entry
     \* @\_list\:      Name of the list in which this entry is placed
     \*
     \* This function returns \`\`(\_type \*)\`\` pointer to the very first entry of a
     \* linker-generated array placed into subsection of \_\_u\_boot\_list section
     \* specified by \_list argument.
     \*
     \* Since this macro defines an array start symbol, its leftmost index
     \* must be 2 and its rightmost index must be 1.
     \*
     \* Example\:
     \*
     \* \:\:
     \*
     \*   struct my\_sub\_cmd \*msc = ll\_entry\_start(struct my\_sub\_cmd, cmd\_sub);
     \*\/
    \#define ll\_entry\_start(\_type, \_list)                                    \\
    (\{                                                                      \\
            static char start[0] \_\_aligned(CONFIG\_LINKER\_LIST\_ALIGN)        \\
                    \_\_attribute\_\_((unused))                                 \\
                    \_\_section("\_\_u\_boot\_list\_2\_"\#\_list"\_1");                        \\
            \_type \* tmp = (\_type \*)\&start;                                  \\
            asm(""\:"+r"(tmp));                                              \\
            tmp;                                                            \\
    \})

    \/\*\*
     \* ll\_entry\_end() - Point after last entry of linker-generated array
     \* @\_type\:      Data type of the entry
     \* @\_list\:      Name of the list in which this entry is placed
     \*              (with underscores instead of dots)
     \*
     \* This function returns \`\`(\_type \*)\`\` pointer after the very last entry of
     \* a linker-generated array placed into subsection of \_\_u\_boot\_list
     \* section specified by \_list argument.
     \*
     \* Since this macro defines an array end symbol, its leftmost index
     \* must be 2 and its rightmost index must be 3.
     \*
     \* Example\:
     \*
     \* \:\:
     \*
     \*   struct my\_sub\_cmd \*msc = ll\_entry\_end(struct my\_sub\_cmd, cmd\_sub);
     \*\/
    \#define ll\_entry\_end(\_type, \_list)                                      \\
    (\{                                                                      \\
            static char end[0] \_\_aligned(4) \_\_attribute\_\_((unused))         \\
                    \_\_section("\_\_u\_boot\_list\_2\_"\#\_list"\_3");                        \\
            \_type \* tmp = (\_type \*)\&end;                                    \\
            asm(""\:"+r"(tmp));                                              \\
            tmp;                                                            \\
    \})
    \/\*\*
     \* ll\_entry\_count() - Return the number of elements in linker-generated array
     \* @\_type\:      Data type of the entry
     \* @\_list\:      Name of the list of which the number of elements is computed
     \*
     \* This function returns the number of elements of a linker-generated array
     \* placed into subsection of \_\_u\_boot\_list section specified by \_list
     \* argument. The result is of an unsigned int type.
     \*
     \* Example\:
     \*
     \* \:\:
     \*
     \*   int i;
     \*   const unsigned int count = ll\_entry\_count(struct my\_sub\_cmd, cmd\_sub);
     \*   struct my\_sub\_cmd \*msc = ll\_entry\_start(struct my\_sub\_cmd, cmd\_sub);
     \*   for (i = 0; i \< count; i++, msc++)
     \*           printf("Entry \%i, x=\%i y=\%i\\n", i, msc-\>x, msc-\>y);
     \*\/
    \#define ll\_entry\_count(\_type, \_list)                                    \\
            (\{                                                              \\
                    \_type \*start = ll\_entry\_start(\_type, \_list);            \\
                    \_type \*end = ll\_entry\_end(\_type, \_list);                \\
                    unsigned int \_ll\_result = end - start;                  \\
                    \_ll\_result;                                             \\
            \})

    \/\*\*
     \* ll\_entry\_get() - Retrieve entry from linker-generated array by name
     \* @\_type\:      Data type of the entry
     \* @\_name\:      Name of the entry
     \* @\_list\:      Name of the list in which this entry is placed
     \*
     \* This function returns a pointer to a particular entry in linker-generated
     \* array identified by the subsection of u\_boot\_list where the entry resides
     \* and it's name.
     \*
     \* Example\:
     \*
     \* \:\:
     \*
     \*   ll\_entry\_declare(struct my\_sub\_cmd, my\_sub\_cmd, cmd\_sub) = \{
     \*           .x = 3,
     \*           .y = 4,
     \*   \};
     \*   ...
     \*   struct my\_sub\_cmd \*c = ll\_entry\_get(struct my\_sub\_cmd, my\_sub\_cmd, cmd\_sub);
     \*\/
    \#define ll\_entry\_get(\_type, \_name, \_list)                               \\
            (\{                                                              \\
                    extern \_type \_u\_boot\_list\_2\_\#\#\_list\#\#\_2\_\#\#\_name;        \\
                    \_type \*\_ll\_result =                                     \\
                            \&\_u\_boot\_list\_2\_\#\#\_list\#\#\_2\_\#\#\_name;            \\
                    \_ll\_result;                                             \\
            \})

    \/\*\*
     \* ll\_entry\_ref() - Get a reference to a linker-generated array entry
     \*
     \* Once an extern ll\_entry\_declare() has been used to declare the reference,
     \* this macro allows the entry to be accessed.
     \*
     \* This is like ll\_entry\_get(), but without the extra code, so it is suitable
     \* for putting into data structures.
     \*
     \* @\_type\: C type of the list entry, e.g. 'struct foo'
     \* @\_name\: name of the entry
     \* @\_list\: name of the list
     \*\/
    \#define ll\_entry\_ref(\_type, \_name, \_list)                               \\
            ((\_type \*)\&\_u\_boot\_list\_2\_\#\#\_list\#\#\_2\_\#\#\_name)

    \/\*\*
     \* ll\_start() - Point to first entry of first linker-generated array
     \* @\_type\:      Data type of the entry
     \*
     \* This function returns \`\`(\_type \*)\`\` pointer to the very first entry of
     \* the very first linker-generated array.
     \*
     \* Since this macro defines the start of the linker-generated arrays,
     \* its leftmost index must be 1.
     \*
     \* Example\:
     \*
     \* \:\:
     \*
     \*   struct my\_sub\_cmd \*msc = ll\_start(struct my\_sub\_cmd);
     \*\/
    \#define ll\_start(\_type)                                                 \\
    (\{                                                                      \\
            static char start[0] \_\_aligned(4) \_\_attribute\_\_((unused))       \\
                    \_\_section("\_\_u\_boot\_list\_1");                           \\
            \_type \* tmp = (\_type \*)\&start;                                  \\
            asm(""\:"+r"(tmp));                                              \\
            tmp;                                                            \\
    \})

    \/\*\*
     \* ll\_end() - Point after last entry of last linker-generated array
     \* @\_type\:      Data type of the entry
     \*
     \* This function returns \`\`(\_type \*)\`\` pointer after the very last entry of
     \* the very last linker-generated array.
     \*
     \* Since this macro defines the end of the linker-generated arrays,
     \* its leftmost index must be 3.
     \*
     \* Example\:
     \*
     \* \:\:
     \*
     \*   struct my\_sub\_cmd \*msc = ll\_end(struct my\_sub\_cmd);
     \*\/
    \#define ll\_end(\_type)                                                   \\
    (\{                                                                      \\
            static char end[0] \_\_aligned(4) \_\_attribute\_\_((unused))         \\
                    \_\_section("\_\_u\_boot\_list\_3");                           \\
            \_type \* tmp = (\_type \*)\&end;                                    \\
            asm(""\:"+r"(tmp));                                              \\
            tmp;                                                            \\
    \})

    \#endif \/\* \_\_ASSEMBLY\_\_ \*\/

    \#endif  \/\* \ :ref:`__LINKER_LISTS_H__ <--linker-lists-h-->` \*\/

# 2 Basic Data Types Atoms

# 2. Basic Data Types: Atoms¶

## 2.0 Overview¶

All data is ultimately built from atoms. An atom is an irreducible value of a specific data type. The basic data types in q correspond mostly to those of traditional programming languages with additional date- and time-related types that facilitate time series. The tables below summarize the basic data types, giving the corresponding types in SQL, Java and C#. Other data types, including enumerations and functions, will be covered when they are discussed in later sections.

Type names

The words under the q heading â boolean, short, int, etc. â are not reserved in q, so they are not displayed in a special font in this text. They do have special meaning when used as name arguments in some operators, so do not use them as names.

The next table collects the important information about q data types. We shall refer to this in subsequent sections.

`1b``0b``0x26``0x00``42h``0Nh``42i``0Ni``42j``0Nj``4.2e``0Ne``4.2``0n``"z"``" "```zaphod`````2015.01.01T00:00:00.000000000``0Np``2006.07m``0Nm``2006.07.21``0Nd``2006.07.21T09:13:39``0Nz``12:00:00.000000000``0Nn``23:59``0Nu``23:59:59``0Nv``09:01:02.042``0Nt```sym$`kx``([] c1:`a`b`c; c2:10 20 30)```a`b`v!10 20 30``{x}``::`## 2.1 Integer Data¶

Integer data types are ubiquitous in programming. There are three integer types in q.

### 2.1.1 long¶

In q versions 3.0 and later, the basic integer type is a signed eight-byte integer, called long. A literal is identified as a long by the fact that it contains only numeric digits, with an optional leading minus sign, and no decimal point. It may also have an optional trailing type indicator j indicating it is a long and not another integer type. Here is a typical long integer value.

`j````q
q)42
42
```

Observe that the type indicator j is accepted but redundant.

`j````q
q)42j
42
```

Default integer type

In q versions 2.8 and earlier the default integer type when no type indicator was given was the four-byte int. Any 2.* long values given with j will carry over to 3.* but if you intend a four-byte value you must now explicitly include the type indicator i.

`j``i`### 2.1.2 short and int¶

The two smaller signed integer data types are short and int. The short type represents a two-byte signed integer and requires the trailing type indicator h. For example,

`h````q
q)-123h
_
```

Similarly, the int type represents a four-byte signed integer and requires the trailing type indicator i.

`i````q
q)1234567890i
_
```

Type promotion

Type promotion is performed automatically in arithmetic operations. However, for a homogenous list of atoms of "wide" type, should a narrower type be presented for update or append in place, the narrow type will not be automatically promoted and an error will result. This may be unintuitive in the context of other type promotion, but it will make sense for table columns.

## 2.2 Floating Point Data¶

Single- and double-precision floating-point data types are supported. Double precision is more common.

### 2.2.1 float¶

The float type represents an IEEE standard eight-byte floating-point number, often called "double" in traditional languages. A float can hold (at least) 15 decimal digits of precision. It is denoted by optionally signed numeric digits with either a decimal point or an optional trailing type indicator f. Observe that the console shortens the display of floats with no significant digits to the right of the decimal.

`f````q
q)3.14159265
_
q)1f
_
q)1.0
_
```

A float can also be specified in scientific notation. Here the e standards for "exponent" â i.e., a power of 10 â and should not be confused with a type indicator. To the right of the e is a two digit signed exponent. The + and leading 0 for a positive exponent are optional.

`e``e``+``0````q
q)1.234e07
_
q)1.234e7
_
q)1.234e-7
1.234e-07
```

### 2.2.2 real¶

The real type represents a single-precision, four-byte floating-point number and is denoted by numeric digits containing a decimal point and a trailing type indicator e. Be mindful that this type is called 'float' in some languages. A real can hold at least 6 decimal digits of precision.

`e`Real datatype

The real type is basically useless in finance since it does not provide enough precision for quantities expressed in currencies such as Yen. We recommend always using float.

The scientific notation of reals is awkward, given the presence of e both for the exponent and the type indicator.

`e````q
q)12.34e
_
q)1.234e7e
_
```

### 2.2.3 Floating Point Display¶

The q console display defaults to seven decimal digits of accuracy for float and real values by rounding the display in the seventh significant digit, even though more digits are stored.

```q
q)f:1.23456789e-10
q)r:1.2345678e-10e
q)f
1.234568e-010
q)r
1.234568e+010e
```

You can change this by using the \P command (note upper case) to specify a display width up to 16 digits. If you issue \P 0 the console will display all 17 decimal digits of the underlying binary representation, although the last digit is unreliable.

`\P``\P 0````q
q)f12:1.23456789012
q)f16:1.234567890123456
q)\P 12
q)f12
1.23456789012
q)f16
1.23456789012
q)\P 16
q)f12
1.23456789012
q)f16
1.234567890123456
q)\P 0
q)1%3
0.33333333333333331
```

## 2.3 Binary Data¶

Binary data can be represented as bit or byte values.

### 2.3.1 boolean¶

The boolean type uses one byte to store a bit and is denoted by the bit value with the trailing type indicator b. There are no keywords for 'true' or 'false', nor are there separate logical operators for booleans.

`b````q
q)0b
_
q)1b
_
```

Binary values are implicitly promoted to unsigned integers when participating in arithmetic expressions or comparisons. For example, the following yields an integer.

```q
q)42+1b
_
```

The following yields a float.

```q
q)3.1415+0b
_
```

The ability of booleans to participate in arithmetic can be useful in eliminating conditionals.

q)flag:1b
q)base:100
q)base+flag*42
142

```q
q)flag:1b
q)base:100
q)base+flag*42
142
```

### 2.3.2 byte¶

The byte type uses one byte to store an unsigned 8-bit value and is denoted by the leading type indicator 0x followed by two hexadecimal digits. Upper or lower case can be used for the alpha hex digits but lower case is customary.

`0x````q
q)0x2a
_
q)0x2A
_
```

As with boolean, a byte participates in arithmetic via type promotion to signed int.

```q
q)1+0x29
_
```

### 2.3.3 GUID¶

The guid type was introduced in q3.0. A GUID (globally unique identifier) is a 16-byte binary value that is unique across time and space (well, nearly so). It is ideally suited for locally generating a globally unique identifier without resorting to a central control mechanism â e.g., transaction IDs. It can be used as a table key or in joins and is preferred to strings or symbols in such situations.

The guid type does not have a literal form since it is generated for you by a process that guarantees uniqueness. Applying ? to the null guid value 0Ng generates a list of guids.

`?``0Ng````q
q)1?0Ng
,61f35174-90bc-a48a-d88f-e15e4a377ec8
q)2?0Ng
_
q)-1?0Ng
_
q)-2?0Ng
_
```

Sign of the left argument

The difference between using a positive integer vs. a negative integer to generate a list of GUIDs is that the positive case uses the same initial seed in each new q session whereas the negative case uses a random seed. The former is useful for reproducible results during testing but only the latter should be used in production; otherwise, your "GUIDs" will not be unique across q sessions.

You can import a guid generated elsewhere by parsing a string of 16 hex digits.

```q
q)"G"$"61f35174-90bc-a48a-d88f-e15e4a377ec8"
_
```

You can also convert from a list of 16 bytes using an overload of sv.

`sv````q
q)0x0 sv 16?0xff
_
```

Unless these values have been constructed from a legitimate GUID creation process, there is no guarantee they will be unique.

The only operations available for guids are ~, =, <, > and null.

`~``=``<``>``null`## 2.4 Text Data¶

There are two atomic text types in q. They are more akin to the SQL types CHAR and VARCHAR than the character types of traditional languages.

`CHAR``VARCHAR`### 2.4.1 char¶

A char holds an individual ASCII or 8-bit Unicode character that is stored in one byte. It corresponds to a SQL CHAR. It is denoted by a single character enclosed in double quotes.

`CHAR````q
q)"q"
_
```

Some keyboard characters â e.g., the double-quote â cannot be entered directly into a char since they have special meaning in the q console. As in C, special characters are escaped with a preceding back-slash \. The console display somewhat confusingly displays the escape, but the following are all actually single characters.

`\````q
q)"\"" / double-quote
"\""
q)"\\" / back-slash
_
q)"\n" / newline
_
q)"\r" / return
_
q)"\t" / horizontal tab
_
```

Also as in C, you can escape any ASCII character by specifying its underlying numeric value as three octal digits.

```q
q)"\142"
"b"
```

### 2.4.2 symbol¶

A symbol is an atom holding text. It is denoted by a leading back-quote, read "back tick" in q-speak.

```q
q)`q
_
q)`zaphod
_
```

Symbols are used for names in q. All names are symbols but not all symbols are names.

A symbol is akin to a SQL VARCHAR, in that it can hold an arbitrary number of characters, but is different in that it is atomic. The char "q" and the symbol `kdb are both atomic entities. A symbol is irreducible, meaning that the individual characters that comprise it are not directly accessible.

`VARCHAR``"q"```kdb`A symbol is not a string. We shall see in Chapter 3 that there is an analogue of strings in q, namely a list of char. While a list of char is a kissing cousin to a symbol, we emphasize that a symbol is not a collection of char. The symbol `a and the char "a" are not the same, as we can see by asking q if they are identical.

``a``"a"````q
q)`a~"a"
0b
```

Advanced

A symbol can include arbitrary text, including text that cannot be directly entered from the console â e.g., embedded blanks and special characters such as back-tick. You can manufacture a symbol from any text by casting the corresponding list of char to a symbol. (You will need to escape special characters into the string.) See Â§6.1.5 for more on casting.

q)`$"A symbol with blanks and `"
`A symbol with blanks and `

```q
q)`$"A symbol with blanks and `"
`A symbol with blanks and `
```

## 2.5 Temporal Data¶

In the real world, time is measured in a system of units determined by calendars and clocks. A calendar measures multiples of days whereas a clock subdivides a day into smaller units. The notion of "telling time" associates a time to a number in the system of units provided by some choice of calendar and clock. We call such a value a temporal type.

Astronomers know that it is most convenient to put the calendar and clock together to have a single measure of time on all scales â i.e., UTC. In common practice, calendar and clock measurements can be made separately where the full scale is not needed.

Our system of calendars and clocks is a hodge-podge based on early astronomy. We measure days by counting rotations of the earth on its axis and years by counting revolutions of the Earth. Months evolved from lunar cycles tracking the changes in the phase of the moon.

Imposed on this is a sexagesimal counting system originated by the ancient Sumerians, who were evidently fond of 12. There were (almost) 360 days in a year and exactly 12 hours of daylight and darkness at equinox. There are 60 minutes in an hour. This was good enough for farmers and astronomers to predict sun and moon cycles 4000 years ago. Increased accuracy of clocks has forced us to adopt leap years, leap seconds and other adjustments to make things come out right. Social and legal customs result in differentiating days into various categories such as business days and holidays. Time zones attempt to make clock time correspond to sunlight and the legislative lunacy of daylight savings time shifts clocks with no actual benefit.

Computer systems have tried to map this mess into the ordered world of bits with varying levels of success. With the advent of nanosecond-based temporal types, time measurement in q is now logical and consistent in that all temporal values are integral counts. These counts are offsets from millennium and midnight, not some point in the 1970s when a system designer realized something had to be done about time.

Q handles time series and relational data in a consistent and efficient manner. It extends the basic SQL date and time data types to facilitate temporal operations, which are minimal in SQL and can be clumsy in traditional languages (e.g., Java's original date library and its time zones).

### 2.5.1 date¶

A date is stored as a four-byte signed integer and is denoted by yyyy.mm.dd, where yyyy represents the year, mm the month and dd the day. The underlying value is the count of days from Jan 1, 2000 â positive for post-millennium and negative for pre.

```q
q)2015.01.01
_
q)2000.01.01=0
1b
q)2000.01.02=1
_
q)1999.12.31=-1
_
```

Since real-world months and days begin at 1 (not zero), January is 01. Leading zeroes in months and days are required; their omission causes an error.

`01````q
q)2015.1.1
'2015.1.1
```

The underlying day count can be obtained by casting.

q)`int$2000.02.01
_

```q
q)`int$2000.02.01
_
```

### 2.5.2 Time Types¶

There are two versions of time, depending on the resolution required. If milliseconds are sufficient, use the time type, which stores the count of milliseconds from midnight in a 32-bit signed integer. It is denoted by hh:mm:ss.uuu where hh represents hours on the 24-hour clock, mm represents minutes, ss represents seconds, and uuu represents milliseconds.

```q
q)12:34:56.789
_
q)12:00:00.000=12*60*60*1000
1b
```

Leading zeroes are required in all constituents of a time value. The underlying millisecond count can be obtained by casting to an int.

```q
q)`int$12:00:00.000
_
```

If milliseconds are not sufficient, use the timespan type, which stores the count of nanoseconds from midnight as a long integer.

It is denoted by 0Dhh:mm:ss.nnnnnnnnn where hh represents hours on the 24-hour clock, mm represents minutes, ss represents seconds, and nnnnnnnnn represents nanoseconds. Observe that the leading 0D is optional.

`0D````q
q)12:34:56.123456789
0D12:34:56.123456789
q)12:34:56.123456 / microseconds become nanos
0D12:34:56.123456000
```

Leading zeroes in constituents are again required.

The underlying nanosecond count can be obtained by casting to a long.

```q
q)`long$12:34:56.123456789
_
```

### 2.5.3 Date-Time Types¶

There are two date-time types. The first is deprecated and should not be used; we include it here in case you encounter it in older q code.

A datetime (deprecated) is the lexical combination of a date and a time, separated by T as in the ISO standard format. A datetime value stores in a float the fractional day count from midnight Jan 1, 2000.

`T````q
q)2000.01.01T12:00:00.000
_
q)2000.01.02T12:00:00.000=1.5
1b
```

The underlying fractional day count can be obtained by casting to float.

```q
q)`float$2000.01.02T12:00:00.000
_
```

Extract the date and time portions from a datetime by casting.

```q
q)`date$2000.01.02T12:00:00.000
_
q)`time$2000.01.02T12:00:00.000
_
```

Do not use a datetime for a key or in a join since the underlying float value is fuzzy and may give unexpected results.

The preferred type is timestamp, which is the lexical combination of a date and a timespan, separated by D. The underlying timestamp value is a long representing the count of nanoseconds since the millennium. Post-millennium is positive and pre- is negative.

`D````q
q)2014.11.22D17:43:40.123456789
_
```

The underlying nanosecond count can be obtained by casting to long.

```q
q)`long$2014.11.22D17:43:40.123456789
_
```

Extract the date and timespan constituents from a timestamp by casting.

```q
q)`date$2014.11.22D17:43:40.123456789
_
q)`timespan$2014.11.22D17:43:40.123456789
_
```

Use a timestamp instead of a datetime for a key column or in a join. Or separate into date and time columns.

### 2.5.4 month¶

The month type is stored as a 32-bit signed integer and is denoted by yyyy.mm with a trailing type indicator m. A month value is the count of months since the beginning of the millennium. Post-milieu is positive and pre is negative.

`m````q
q)2015.11m
_
q)2001.01m=12
1b
```

Leaving off the type indicator m yields a float. This is a common qbie error.

`m````q
q)2014.11 / this is a float!
 2014.11
```

The underlying month count can be obtained by casting to int.

```q
q)`int$2015.01m
_
```

Despite that fact that a month type counts months since the millennium and a date type counts days since the millennium, the first day of the month is equal to the month.

q)2015.07m=2015.07.01
1b

```q
q)2015.07m=2015.07.01
1b
```

### 2.5.5 minute¶

The minute type is stored as a 32-bit signed integer and is denoted by hh:mm. A minute value counts the number of minutes from midnight.

```q
q)12:30
_
q)12:00=12*60
1b
```

The underlying minute count can be obtained by casting to int.

```q
q)`int$12:00
_
```

A minute equals its equivalent time and timestamp counterparts.

```q
q)12:00=12:00:00.000
1b
q)12:00=12:00:00.000000000
_
```

### 2.5.6 second¶

The second type is stored as 32-bit signed integer and is denoted by hh:mm:ss. A second value counts the number of seconds from midnight.

```q
q)23:59:59
_
q)23:59:59=-1+24*60*60
1b
```

The representation of the second type makes it look like an ordinary time value, and it can function as that if you only need resolution to the second. However, a q time value is a count of milliseconds or nanoseconds from midnight, so the underlying values are different.

```q
q)`int$12:34:56
45296i
q)`int$12:34:56.000
_
q)`long$12:34:56.000000000
_
```

Nevertheless, these values are equal in the eyes of q â as they should be, since they are merely representations in different units of the same position on a clock.

```q
q)12:34:56=12:34:56.000
1b
q)12:34:56.000=12:34:56.000000000
_
```

### 2.5.7 Constituents and Dot Notation¶

The constituents of compound temporal types can be extracted using dot notation. For example, the field values of a date are named year, mm and dd; similarly for time and other temporal types.

`year``mm``dd````q
q)dt:2014.01.01
q)dt.year
2014i
q)dt.mm
_
q)dt.dd
_
q)ti:12:34:56.789
q)ti.hh
12i
q)ti.mm
_
q)ti.ss
_
```

Unfortunately, at the time of this writing (Sep 2015) dot notation for extraction (still) does not work inside functions.

Thus we recommend avoiding dot notation altogether and using Cast instead, as it always works for any meaningful temporal extraction or conversion. In addition to the individual field values, you can also extract higher-order constituents.

```q
q)`dd$dt
1i
q)`mm$dt
_
q)`dd$dt
_
q)`month$dt
2014.01m
```

To extract milliseconds or nanoseconds from a time type, cast to the underlying integer and mod the result by 1000 or 1000000000.

`mod`q)(`int$12:34:56.789) mod 1000
789
q)(`long$12:34:56.123456789) mod 1000000000
_

```q
q)(`int$12:34:56.789) mod 1000
789
q)(`long$12:34:56.123456789) mod 1000000000
_
```

## 2.6 Arithmetic Infinities and Nulls¶

Types whose underlying values are integer or floating point have special lexical forms representing values that lie outside the "normal" domain. We list the basic ones in the following table. The others are obtained by appending the appropriate type suffices.

Observe the distinction between lower case w in the float literals and upper case W in the integer literals. The character w was chosen for its resemblance to the infinity symbol. Seriously.

`w``W``w`In q, division of numeric values always results in a float.

In mathematics, division of a positive value by 0 results in positive infinity and division of a negative value by zero results in negative infinity. So it is in q, with the funky symbols 0w and -0w for positive and negative float infinity respectively.

`0w``-0w`In mathematics, division of zero by zero is undefined. So it is in q, with 0n representing NaN â i.e., an undefined float. The float infinities perform exactly as they should in arithmetic and comparison operations, since they are required to do so by the IEEE spec.

`0n`The q philosophy is that any valid arithmetic expression will produce a result rather than a runtime error. Therefore, dividing by 0 produces a special float value rather than throwing an exception. You can perform a complex sequence of calculations without worrying about things blowing up in the middle or having to insert cumbersome exception trapping.

The integral infinities and nulls cannot be produced via division on normal integer values, since the result of division in q is always a float. Moreover, while integral nulls propagate as nulls should, the integral infinities do not perform as you would expect in arithmetic operations. The integral infinities do produce the correct results in comparisons; in fact, this is their raison dâÃªtre.

```q
q)42<0W
1b
q)-0W<42
_
```

To understand the integral nulls and infinities, realize that they are actually valid bit patterns for their corresponding types. Here are the long versions.

Consequently, ordering on integers is,

0N < -0W < normal integer < 0W

`0N``-0W``0W`This explains some oddities.

```q
q)9223372036854775806+1
0W
q)-0W-1
0N
q)-0W+1
-9223372036854775806
```

Q does not trap overflow

The fact that q does not trap overflow explains the equally bizarre looking

q)0W+1
0N
q)0W+2
-0W
q)0W+3
-9223372036854775806

```q
q)0W+1
0N
q)0W+2
-0W
q)0W+3
-9223372036854775806
```

Implementing proper arithmetic on integer infinities would entail expensive tests in the arithmetic operators and an unacceptable slow-down for normal arithmetic.

## 2.7 Nulls¶

### 2.7.0 Overview of Nulls¶

The concept of a null value generally indicates missing data. This is an area in which q differs from both traditional programming languages and SQL.

In such languages as C++, Java and C#, the concept of a null value applies to complex entities (i.e., objects) that are allocated on the heap and accessed by pointer or reference. A null pointer corresponds to an unallocated entity, meaning that it has not been assigned the address of an initialized block of memory. (Tony Hoare, who introduced the concept of null pointer, calls it his "billion dollar mistake.") There is no concept of null for entities that are of value type. For those types that admit null, you test for null by asking if the value is equal to a special null marker.

The NULL value in SQL indicates that data is not present. The NULL value is distinct from any value that can actually be contained in a field and it does not have '=' semantics. That is, you do not test a field for null with = NULL. Instead, you ask if it IS NULL. Because NULL is a separate value, Boolean fields, for example, actually have three states: 0, 1 and NULL.

`NULL``NULL``= NULL``IS NULL``NULL``0``1``NULL`The q situation is more interesting. There are no references or pointers, so the notion of an unallocated entity does not arise. Most types have null values that are distinct from "normal" values and occupy the same amount of storage. Some types do not designate a distinct null value because there is no available bit pattern â i.e., for boolean, byte and char all underlying bit patterns are meaningfully employed. In this case, the value with no information content serves as a proxy for null.

The following table summarizes the way nulls are handled.

### 2.7.1 Binary Nulls¶

The binary types have no null values. There is no room since every bit pattern is a legitimate value.

### 2.7.2 Numeric and Temporal Nulls¶

The numeric and temporal types have their own designated null values. Here the situation is similar to SQL, in that you can distinguish missing data from data whose underlying value is zero. In contrast, there is no universal null value and q nulls take the same space as non-nulls.

An advantage of the q approach is that the null values act like other values in expressions. The tradeoff is that you must use the correct null value in type-checked situations.

### 2.7.3 Text Nulls¶

Considering a symbol as variable length text justifies that the symbol null is the empty symbol, designated by a naked back-tick `.

```The null value for the char type is the blank character " ". As with binary data, you cannot distinguish between a missing char value and a blank value. Again, this is not seriously limiting in practice, but you should ensure that your application does not rely on this distinction.

`" "`The value "" is not a null char. It is an empty list of char.

`""`### 2.7.4 Testing for Null¶

You could test for null using = but this requires a null literal of correct type. Because q is dynamically typed, this can result in problems if a variable changes type during program execution.

`=`Always use the unary null to test a value for null, as opposed to =, as it provides a type-independent check. Also, you don't have to remember the funky null literals.

`null``=````q
q)null 42
_
q)null `
_
q)null " "
_
q)null ""
```


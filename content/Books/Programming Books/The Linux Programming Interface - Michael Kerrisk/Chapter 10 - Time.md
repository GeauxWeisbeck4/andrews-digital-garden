---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 10 - Time
modified: 2024-11-11T19:08:56-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **10**  
**TIME**

Within a program, we may be interested in two kinds of time:

• _Real time_: This is the time as measured either from some standard point (_calendar_ time) or from some fixed point (typically the start) in the life of a process (_elapsed_ or _wall clock_ time). Obtaining the calendar time is useful to programs that, for example, timestamp database records or files. Measuring elapsed time is useful for a program that takes periodic actions or makes regular measurements from some external input device.

• _Process time_: This is the amount of CPU time used by a process. Measuring process time is useful for checking or optimizing the performance of a program or algorithm.

Most computer architectures have a built-in hardware clock that enables the kernel to measure real and process time. In this chapter, we look at system calls for dealing with both sorts of time, and library functions for converting between human-readable and internal representations of time. Since human-readable representations of time are dependent on the geographical location and on linguistic and cultural conventions, discussion of these representations leads us into an investigation of timezones and locales.

### **10.1 Calendar Time**

Regardless of geographic location, UNIX systems represent time internally as a measure of seconds since the Epoch; that is, since midnight on the morning of 1 January 1970, Coordinated Universal Time (UTC, previously known as Greenwich Mean Time, or GMT). This is approximately the date when the UNIX system came into being. Calendar time is stored in variables of type _time_t_, an integer type specified by SUSv3.

On 32-bit Linux systems, _time_t_, which is a signed integer, can represent dates in the range 13 December 1901 20:45:52 to 19 January 2038 03:14:07. (SUSv3 leaves the meaning of negative _time_t_ values unspecified.) Thus, many current 32-bit UNIX systems face a theoretical _Year 2038_ problem, which they may encounter before 2038, if they do calculations based on dates in the future. This problem will be significantly alleviated by the fact that by 2038, probably all UNIX systems will have long become 64-bit and beyond. However, 32-bit embedded systems, which typically have a much longer lifespan than desktop hardware, may still be afflicted by the problem. Furthermore, the problem will remain for any legacy data and applications that maintain time in a 32-bit _time_t_ format.

The _gettimeofday()_ system call returns the calendar time in the buffer pointed to by _tv_.

#include <sys/time.h>  
  
int gettimeofday(struct timeval *tv, struct timezone *tz);

Returns 0 on success, or –1 on error

The _tv_ argument is a pointer to a structure of the following form:

struct timeval {  
    time_t      tv_sec;     /* Seconds since 00:00:00, 1 Jan 1970 UTC */  
    suseconds_t tv_usec;    /* Additional microseconds (long int) */  
};

Although the _tv_usec_ field affords microsecond precision, the accuracy of the value it returns is determined by the architecture-dependent implementation. (The _u_ in _tv_usec_ derives from the resemblance to the Greek letter © (“mu”) used in the metric system to denote one-millionth.) On modern x86-32 systems (i.e., Pentium systems with a Timestamp Counter register that is incremented once at each CPU clock cycle), _gettimeofday()_ does provide microsecond accuracy.

The _tz_ argument to _gettimeofday()_ is a historical artifact. In older UNIX implementations, it was used to retrieve timezone information for the system. This argument is now obsolete and should always be specified as NULL. (SUSv4 marks _gettimeofday()_ obsolete, presumably in favor of the POSIX clocks API described in [Section 23.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch23.xhtml#ch23lev1sec05).)

If the _tz_ argument is supplied, then it returns a _timezone_ structure whose fields contain whatever values were specified in the (obsolete) _tz_ argument in a previous call to _settimeofday()_. This structure contains two fields: _tz_minuteswest_ and _tz_dsttime_. The _tz_minuteswest_ field indicates the number of minutes that must be added to times in this zone to match UTC, with a negative value indicating an adjustment of minutes to the east of UTC (e.g., for Central European Time, one hour ahead of UTC, this field would contain the value –60). The _tz_dsttime_ field contains a constant that was designed to represent the daylight saving time (DST) regime in force in this timezone. It is because the DST regime can’t be represented using a simple algorithm that the _tz_ argument is obsolete. (This field has never been supported on Linux.) See the _gettimeofday(2)_ manual page for further details.

The _time()_ system call returns the number of seconds since the Epoch (i.e., the same value that _gettimeofday()_ returns in the _tv_sec_ field of its _tv_ argument).

#include <time.h>  
  
time_t time(time_t *timep);

Returns number of seconds since the Epoch, or _(time_t)_ –1 on error

If the _timep_ argument is not NULL, the number of seconds since the Epoch is also placed in the location to which _timep_ points.

Since _time()_ returns the same value in two ways, and the only possible error that can occur when using _time()_ is to give an invalid address in the _timep_ argument (EFAULT), we often simply use the following call (without error checking):

t = time(NULL);

The reason for the existence of two system calls (_time()_ and _gettimeofday()_) with essentially the same purpose is historical. Early UNIX implementations provided _time()_. 4.2BSD added the more precise _gettimeofday()_ system call. The existence of _time()_ as a system call is now redundant; it could be implemented as a library function that calls _gettimeofday()_.

### **10.2 Time-Conversion Functions**

[Figure 10-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10fig1) shows the functions used to convert between _time_t_ values and other time formats, including printable representations. These functions shield us from the complexity brought to such conversions by timezones, daylight saving time (DST) regimes, and localization issues. (We describe timezones in [Section 10.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec03) and locales in [Section 10.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec04).)

SUSv4 marks _ctime()_ and _asctime()_ obsolete, because they do not return localized strings (and they are nonreentrant).

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f10-01.jpg)

**Figure 10-1:** Functions for retrieving and working with calendar time

#### **10.2.1 Converting _time_t_ to Printable Form**

The _ctime()_ function provides a simple method of converting a _time_t_ value into printable form.

#include <time.h>  
  
char *ctime(const time_t *timep);

Returns pointer to statically allocated string terminated by newline and \0 on success, or NULL on error

Given a pointer to a _time_t_ value in _timep_, _ctime()_ returns a 26-byte string containing the date and time in a standard format, as illustrated by the following example:

Wed Jun 8 14:22:34 2011

The string includes a terminating newline character and a terminating null byte. The _ctime()_ function automatically accounts for local timezone and DST settings when performing the conversion. (We explain how these settings are determined in [Section 10.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec03).) The returned string is statically allocated; future calls to _ctime()_ will overwrite it.

SUSv3 states that calls to any of the functions _ctime()_, _gmtime()_, _localtime()_, or _asctime()_ may overwrite the statically allocated structure that is returned by any of the other functions. In other words, these functions may share single copies of the returned character array and _tm_ structure, and this is done in some versions of _glibc_. If we need to maintain the returned information across multiple calls to these functions, we must save local copies.

A reentrant version of _ctime()_ is provided in the form of _ctime_r()_. (We explain reentrancy in [Section 21.1.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec02).) This function permits the caller to specify an additional argument that is a pointer to a (caller-supplied) buffer that is used to return the time string. Other reentrant versions of functions mentioned in this chapter operate similarly.

#### **10.2.2 Converting Between _time_t_ and Broken-Down Time**

The _gmtime()_ and _localtime()_ functions convert a _time_t_ value into a so-called _broken-down time_. The broken-down time is placed in a statically allocated structure whose address is returned as the function result.

#include <time.h>  
  
struct tm *gmtime(const time_t *timep);  
struct tm *localtime(const time_t *timep);

Both return a pointer to a statically allocated broken-down time structure on success, or NULL on error

The _gmtime()_ function converts a calendar time into a broken-down time corresponding to UTC. (The letters _gm_ derive from Greenwich Mean Time.) By contrast, _localtime()_ takes into account timezone and DST settings to return a broken-down time corresponding to the system’s local time.

Reentrant versions of these functions are provided as _gmtime_r()_ and _localtime_r()_.

The _tm_ structure returned by these functions contains the date and time fields broken into individual parts. This structure has the following form:

struct tm {  
    int tm_sec;         /* Seconds (0-60) */  
    int tm_min;         /* Minutes (0-59) */  
    int tm_hour;        /* Hours (0-23) */  
    int tm_mday;        /* Day of the month (1-31) */  
    int tm_mon;         /* Month (0-11) */  
    int tm_year;        /* Year since 1900 */  
    int tm_wday;        /* Day of the week (Sunday = 0)*/  
    int tm_yday;        /* Day in the year (0-365; 1 Jan = 0)*/  
    int tm_isdst;       /* Daylight saving time flag  
                             > 0: DST is in effect;  
                             = 0: DST is not effect;  
                             < 0: DST information not available */  
};

The _tm_sec_ field can be up to 60 (rather than 59) to account for the leap seconds that are occasionally applied to account for the difference between the precise International Atomic Time (TAI) and the observed solar time, which is less precise because of factors such as geological events and the gradual slowdown in the earth’s rotation.

If the _BSD_SOURCE feature test macro is defined, then the _glibc_ definition of the _tm_ structure also includes two additional fields containing further information about the represented time. The first of these, _long int tm_gmtoff_, contains the number of seconds that the represented time falls east of UTC. The second field, _const char *tm_zone_, is the abbreviated timezone name (e.g., _CEST_ for Central European Summer Time). SUSv3 doesn’t specify either of these fields, and they appear on only a few other UNIX implementations (mainly BSD derivatives).

The _mktime()_ function translates a broken-down time, expressed as local time, into a _time_t_ value, which is returned as the function result. The caller supplies the broken-down time in a _tm_ structure pointed to by _timeptr_. During this translation, the _tm_wday_ and _tm_yday_ fields of the input _tm_ structure are ignored.

#include <time.h>  
  
time_t mktime(struct tm *timeptr);

Returns seconds since the Epoch corresponding to _timeptr_ on success, or _(time_t)_ –1 on error

The _mktime()_ function may modify the structure pointed to by _timeptr_. At a minimum, it ensures that the _tm_wday_ and _tm_yday_ fields are set to values that correspond appropriately to the values of the other input fields.

In addition, _mktime()_ doesn’t require the other fields of the _tm_ structure to be restricted to the ranges described earlier. For each field whose value is out of range, _mktime()_ adjusts that field’s value so that it is in range and makes suitable adjustments to the other fields. All of these adjustments are performed before _mktime()_ updates the _tm_wday_ and _tm_yday_ fields and calculates the returned _time_t_ value.

For example, if the input _tm_sec_ field were 123, then on return, the value of this field would be 3, and the value of the _tm_min_ field would have 2 added to whatever value it previously had. (And if that addition caused _tm_min_ to overflow, then the _tm_min_ value would be adjusted and the _tm_hour_ field would be incremented, and so on.) These adjustments even apply for negative field values. For example, specifying –1 for _tm_sec_ means the 59th second of the previous minute. This feature is useful since it allows us to perform date and time arithmetic on a broken-down time value.

The timezone setting is used by _mktime()_ when performing the translation. In addition, the DST setting may or may not be used, depending on the value of the input _tm_isdst_ field:

• If _tm_isdst_ is 0, treat this time as standard time (i.e., ignore DST, even if it would be in effect at this time of year).

• If _tm_isdst_ is greater than 0, treat this time as DST (i.e., behave as though DST is in effect, even if it would not normally be so at this time of year).

• If _tm_isdst_ is less than 0, attempt to determine if DST would be in effect at this time of the year. This is typically the setting we want.

On completion (and regardless of the initial setting of _tm_isdst_), _mktime()_ sets the _tm_isdst_ field to a positive value if DST is in effect at the given date, or to 0 if DST is not in effect.

#### **10.2.3 Converting Between Broken-Down Time and Printable Form**

In this section, we describe functions that convert a broken-down time to printable form, and vice versa.

##### **Converting from broken-down time to printable form**

Given a pointer to a broken-down time structure in the argument _tm_, _asctime()_ returns a pointer to a statically allocated string containing the time in the same form as _ctime()_.

#include <time.h>  
  
char *asctime(const struct tm *timeptr);

Returns pointer to statically allocated string terminated by newline and \0 on success, or NULL on error

By contrast with _ctime()_, local timezone settings have no effect on _asctime()_, since it is converting a broken-down time that is typically either already localized via _localtime()_ or in UTC as returned by _gmtime()_.

As with _ctime()_, we have no control over the format of the string produced by _asctime()_.

A reentrant version of _asctime()_ is provided in the form of _asctime_r()_.

[Listing 10-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10ex1) demonstrates the use of _asctime()_, as well as all of the time-conversion functions described so far in this chapter. This program retrieves the current calendar time, and then uses various time-conversion functions and displays their results. Here is an example of what we see when running this program in Munich, Germany, which (in winter) is on Central European Time, one hour ahead of UTC:

$ date  
Tue Dec 28 16:01:51 CET 2010  
$ ./calendar_time  
Seconds since the Epoch (1 Jan 1970): 1293548517 (about 40.991 years)  
  gettimeofday() returned 1293548517 secs, 715616 microsecs  
Broken down by gmtime():  
  year=110 mon=11 mday=28 hour=15 min=1 sec=57 wday=2 yday=361 isdst=0  
Broken down by localtime():  
  year=110 mon=11 mday=28 hour=16 min=1 sec=57 wday=2 yday=361 isdst=0  
  
asctime() formats the gmtime() value as: Tue Dec 28 15:01:57 2010  
ctime() formats the time() value as:     Tue Dec 28 16:01:57 2010  
mktime() of gmtime() value:    1293544917 secs  
mktime() of localtime() value: 1293548517 secs      3600 secs ahead of UTC

**Listing 10-1:** Retrieving and converting calendar times

_____________________________________________________ time/calendar_time.c  
  
#include <locale.h>  
#include <time.h>  
#include <sys/time.h>  
#include "tlpi_hdr.h"  
#define SECONDS_IN_TROPICAL_YEAR (365.24219 * 24 * 60 * 60)  
  
int  
main(int argc, char *argv[])  
{  
    time_t t;  
    struct tm *gmp, *locp;  
    struct tm gm, loc;  
    struct timeval tv;  
  
    t = time(NULL);  
    printf("Seconds since the Epoch (1 Jan 1970): %ld", (long) t);  
    printf(" (about %6.3f years)\n", t / SECONDS_IN_TROPICAL_YEAR);  
  
    if (gettimeofday(&tv, NULL) == -1)  
        errExit("gettimeofday");  
    printf("  gettimeofday() returned %ld secs, %ld microsecs\n",  
            (long) tv.tv_sec, (long) tv.tv_usec);  
  
    gmp = gmtime(&t);  
    if (gmp == NULL)  
        errExit("gmtime");  
  
    gm = *gmp;          /* Save local copy, since *gmp may be modified  
                           by asctime() or gmtime() */  
    printf("Broken down by gmtime():\n");  
    printf("  year=%d mon=%d mday=%d hour=%d min=%d sec=%d ", gm.tm_year,  
            gm.tm_mon, gm.tm_mday, gm.tm_hour, gm.tm_min, gm.tm_sec);  
    printf("wday=%d yday=%d isdst=%d\n", gm.tm_wday, gm.tm_yday, gm.tm_isdst);  
  
    locp = localtime(&t);  
    if (locp == NULL)  
        errExit("localtime");  
  
    loc = *locp;        /* Save local copy */  
  
    printf("Broken down by localtime():\n");  
    printf("  year=%d mon=%d mday=%d hour=%d min=%d sec=%d ",  
            loc.tm_year, loc.tm_mon, loc.tm_mday,  
            loc.tm_hour, loc.tm_min, loc.tm_sec);  
    printf("wday=%d yday=%d isdst=%d\n\n",  
            loc.tm_wday, loc.tm_yday, loc.tm_isdst);  
  
    printf("asctime() formats the gmtime() value as: %s", asctime(&gm));  
    printf("ctime() formats the time() value as:     %s", ctime(&t));  
  
    printf("mktime() of gmtime() value:    %ld secs\n", (long) mktime(&gm));  
    printf("mktime() of localtime() value: %ld secs\n", (long) mktime(&loc));  
  
    exit(EXIT_SUCCESS);  
}  
_____________________________________________________ time/calendar_time.c

The _strftime()_ function provides us with more precise control when converting a broken-down time into printable form. Given a broken-down time pointed to by _timeptr_, _strftime()_ places a corresponding null-terminated, date-plus-time string in the buffer pointed to by _outstr_.

#include <time.h>  
  
size_t strftime(char *outstr, size_t maxsize, const char *format,  
                const struct tm *timeptr);

Returns number of bytes placed in _outstr_ (excluding terminating null byte) on success, or 0 on error

The string returned in _outstr_ is formatted according to the specification in _format_. The _maxsize_ argument specifies the maximum space available in _outstr_. Unlike _ctime()_ and _asctime()_, _strftime()_ doesn’t include a newline character at the end of the string (unless one is included in _format_).

On success, _strftime()_ returns the number of bytes placed in _outstr_, excluding the terminating null byte. If the total length of the resulting string, including the terminating null byte, would exceed _maxsize_ bytes, then _strftime()_ returns 0 to indicate an error, and the contents of _outstr_ are indeterminate.

The _format_ argument to _strftime()_ is a string akin to that given to _printf()_. Sequences beginning with a percent character (%) are conversion specifications, which are replaced by various components of the date and time according to the specifier character following the percent character. A rich set of conversion specifiers is provided, a subset of which is listed in [Table 10-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10table1). (For a complete list, see the _strftime(3)_ manual page.) Except as otherwise noted, all of these conversion specifiers are standardized in SUSv3.

The %U and %W specifiers both produce a week number in the year. The %U week numbers are calculated such that the first week containing a Sunday is numbered 1, and the partial week preceding that is numbered 0. If Sunday happens to fall as the first day of the year, then there is no week 0, and the last day of the year falls in week 53. The %W week numbers work in the same way, but with Monday rather than Sunday.

Often, we want to display the current time in various demonstration programs in this book. For this purpose we provide the function _currTime()_, which returns a string containing the current time as formatted by _strftime()_ when given the argument _format_.

The _currTime()_ function implementation is shown in [Listing 10-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10ex2).

#include "curr_time.h"  
  
char *currTime(const char *format);

Returns pointer to statically allocated string, or NULL on error

**Table 10-1:** Selected conversion specifiers for _strftime()_

|**Specifier**|**Description**|**Example**|
|---|---|---|
|%%|A % character|%|
|%a|Abbreviated weekday name|Tue|
|%A|Full weekday name|Tuesday|
|%b, %h|Abbreviated month name|Feb|
|%B|Full month name|February|
|%c|Date and time|Tue Feb  1 21:39:46 2011|
|%d|Day of month (2 digits, 01 to 31)|01|
|%D|American date (same as %m/%d/%y)|02/01/11|
|%e|Day of month (2 characters)|1|
|%F|ISO date (same as %Y-%m-%d)|2011-02-01|
|%H|Hour (24-hour clock, 2 digits)|21|
|%I|Hour (12-hour clock, 2 digits)|09|
|%j|Day of year (3 digits, 001 to 366)|032|
|%m|Decimal month (2 digits, 01 to 12)|02|
|%M|Minute (2 digits)|39|
|%p|AM/PM|PM|
|%P|am/pm (GNU extension)|pm|
|%R|24-hour time (same as %H:%M)|21:39|
|%S|Second (00 to 60)|46|
|%T|Time (same as %H:%M:%S)|21:39:46|
|%u|Weekday number (1 to 7, Monday = 1)|2|
|%U|Sunday week number (00 to 53)|05|
|%w|Weekday number (0 to 6, Sunday = 0)|2|
|%W|Monday week number (00 to 53)|05|
|%x|Date (localized)|02/01/11|
|%X|Time (localized)|21:39:46|
|%y|2-digit year|11|
|%Y|4-digit year|2011|
|%Z|Timezone name|CET|

**Listing 10-2:** A function that returns a string containing the current time

__________________________________________________________ time/curr_time.c  
  
#include <time.h>  
#include "curr_time.h"           /* Declares function defined here */  
  
#define BUF_SIZE 1000  
  
/* Return a string containing the current time formatted according to  
   the specification in 'format' (see strftime(3) for specifiers).  
   If 'format' is NULL, we use "%c" as a specifier (which gives the  
   date and time as for ctime(3), but without the trailing newline).  
   Returns NULL on error. */  
  
char *  
currTime(const char *format)  
{  
    static char buf[BUF_SIZE];  /* Nonreentrant */  
    time_t t;  
    size_t s;  
    struct tm *tm;  
  
    t = time(NULL);  
    tm = localtime(&t);  
    if (tm == NULL)  
        return NULL;  
  
    s = strftime(buf, BUF_SIZE, (format != NULL) ? format : "%c", tm);  
  
    return (s == 0) ? NULL : buf;  
}  
__________________________________________________________ time/curr_time.c

##### **Converting from printable form to broken-down time**

The _strptime()_ function is the converse of _strftime()_. It converts a date-plus-time string to a broken-down time.

#define _XOPEN_SOURCE  
#include <time.h>  
  
char *strptime(const char *str, const char *format, struct tm *timeptr);

Returns pointer to next unprocessed character in _str_ on success, or NULL on error

The _strptime()_ function uses the specification given in _format_ to parse the date-plus-time string given in _str_, and places the converted broken-down time in the structure pointed to by _timeptr_.

On success, _strptime()_ returns a pointer to the next unprocessed character in _str_. (This is useful if the string contains further information to be processed by the calling program.) If the complete format string could not be matched, _strptime()_ returns NULL to indicate the error.

The format specification given to _strptime()_ is akin to that given to _scanf(3)_. It contains the following types of characters:

• conversion specifications beginning with a percent character (%);

• white-space characters, which match zero or more white spaces in the input string; and

• non-white-space characters (other than %), which must match exactly the same characters in the input string.

The conversion specifications are similar to those given to _strftime()_ ([Table 10-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10table1)). The major difference is that the specifiers are more general. For example, both %a and %A can accept a weekday name in either full or abbreviated form, and %d or %e can be used to read a day of the month with or without a leading 0 in the case of single-digit days. In addition, case is ignored; for example, _May_ and _MAY_ are equivalent month names. The string %% is used to match a percent character in the input string. The _strptime(3)_ manual page provides more details.

The _glibc_ implementation of _strptime()_ doesn’t modify those fields of the _tm_ structure that are not initialized by specifiers in _format_. This means that we can employ a series of _strptime()_ calls to construct a single _tm_ structure from information in multiple strings, such as a date string and a time string. While SUSv3 permits this behavior, it doesn’t require it, and so we can’t rely on it on other UNIX implementations. In a portable application, we must ensure that _str_ and _format_ contain input that will set all fields of the resulting _tm_ structure, or make sure that the _tm_ structure is suitably initialized before calling _strptime()_. In most cases, it would be sufficient to zero out the entire structure using _memset()_, but be aware that a value of 0 in the _tm_mday_ field corresponds to the last day of the previous month in _glibc_ and many other implementations of the time-conversion functions. Finally, note that _strptime()_ never sets the value of the _tm_isdst_ field of the _tm_ structure.

The GNU C library also provides two other functions that serve a similar purpose to _strptime()_: _getdate()_ (specified in SUSv3 and widely available) and its reentrant analog _getdate_r()_ (not specified in SUSv3 and available on only a few other UNIX implementations). We don’t describe these functions here, because they employ an external file (identified by the environment variable DATEMSK) to specify the format used for scanning the date, which makes them somewhat awkward to use and also creates security vulnerabilities in set-user-ID programs.

[Listing 10-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10ex3) demonstrates the use of _strptime()_ and _strftime()_. This program takes a command-line argument containing a date and time, converts this to a broken-down time using _strptime()_, and then displays the result of performing the reverse conversion using _strftime()_. The program takes up to three arguments, of which the first two are required. The first argument is the string containing a date and time. The second argument is the format specification to be used by _strptime()_ to parse the first argument. The optional third argument is the format string to be used by _strftime()_ for the reverse conversion. If this argument is omitted, a default format string is used. (We describe the _setlocale()_ function used in this program in [Section 10.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec04).) The following shell session log shows some examples of the use of this program:

$ ./strtime "9:39:46pm 1 Feb 2011" "%I:%M:%S%p %d %b %Y"  
calendar time (seconds since Epoch): 1296592786  
strftime() yields: 21:39:46 Tuesday, 01 February 2011 CET

The following usage is similar, but this time we explicitly specify a format for _strftime()_:

$ ./strtime "9:39:46pm 1 Feb 2011" "%I:%M:%S%p %d %b %Y" "%F %T"  
calendar time (seconds since Epoch): 1296592786  
strftime() yields: 2011-02-01 21:39:46

**Listing 10-3:** Retrieving and converting calendar times

___________________________________________________________ time/strtime.c  
  
#define _XOPEN_SOURCE  
#include <time.h>  
#include <locale.h>  
#include "tlpi_hdr.h"  
  
#define SBUF_SIZE 1000  
  
int  
main(int argc, char *argv[])  
{  
    struct tm tm;  
    char sbuf[SBUF_SIZE];  
    char *ofmt;  
  
    if (argc < 3 || strcmp(argv[1], "--help") == 0)  
        usageErr("%s input-date-time in-format [out-format]\n", argv[0]);  
  
    if (setlocale(LC_ALL, "") == NULL)  
        errExit("setlocale");   /* Use locale settings in conversions */  
  
    memset(&tm, 0, sizeof(struct tm));          /* Initialize 'tm' */  
    if (strptime(argv[1], argv[2], &tm) == NULL)  
        fatal("strptime");  
  
    tm.tm_isdst = -1;           /* Not set by strptime(); tells mktime()  
                                   to determine if DST is in effect */  
    printf("calendar time (seconds since Epoch): %ld\n", (long) mktime(&tm));  
  
    ofmt = (argc > 3) ? argv[3] : "%H:%M:%S %A, %d %B %Y %Z";  
    if (strftime(sbuf, SBUF_SIZE, ofmt, &tm) == 0)  
        fatal("strftime returned 0");  
    printf("strftime() yields: %s\n", sbuf);  
  
    exit(EXIT_SUCCESS);  
}  
___________________________________________________________ time/strtime.c

### **10.3 Timezones**

Different countries (and sometimes even different regions within a single country) operate on different timezones and DST regimes. Programs that input and output times must take into account the timezone and DST regime of the system on which they are run. Fortunately, all of the details are handled by the C library.

##### **Timezone definitions**

Timezone information tends to be both voluminous and volatile. For this reason, rather than encoding it directly into programs or libraries, the system maintains this information in files in standard formats.

These files reside in the directory /usr/share/zoneinfo. Each file in this directory contains information about the timezone regime in a particular country or region. These files are named according to the timezone they describe, so we may find files with names such as EST (US Eastern Standard Time), CET (Central European Time), UTC, Turkey, and Iran. In addition, subdirectories can be used to hierarchically group related timezones. Under a directory such as Pacific, we may find the files Auckland, Port_Moresby, and Galapagos. When we specify a timezone for use by a program, in effect, we are specifying a relative pathname for one of the timezone files in this directory.

The local time for the system is defined by the timezone file /etc/localtime, which is often linked to one of the files in /usr/share/zoneinfo.

The format of timezone files is documented in the _tzfile(5)_ manual page. Timezone files are built using _zic(8)_, the _zone information compiler_. The _zdump(8)_ command can be used to display the time as it would be currently according to the timezone in a specified timezone file.

##### **Specifying the timezone for a program**

To specify a timezone when running a program, we set the TZ environment variable to a string consisting of a colon (:) followed by one of the timezone names defined in /usr/share/zoneinfo. Setting the timezone automatically influences the functions _ctime()_, _localtime()_, _mktime()_, and _strftime()_.

To obtain the current timezone setting, each of these functions uses _tzset(3)_, which initializes three global variables:

char *tzname[2];    /* Name of timezone and alternate (DST) timezone */  
int daylight;       /* Nonzero if there is an alternate (DST) timezone */  
long timezone;      /* Seconds difference between UTC and local  
                       standard time */

The _tzset()_ function first checks the TZ environment variable. If this variable is not set, then the timezone is initialized to the default defined in the timezone file /etc/localtime. If the TZ environment variable is defined with a value that can’t be matched to a timezone file, or it is an empty string, then UTC is used. The TZDIR environment variable (a nonstandard GNU-extension) can be set to the name of a directory in which timezone information should be sought instead of in the default /usr/share/zoneinfo.

We can see the effect of the TZ variable by running the program in [Listing 10-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10ex4). In the first run, we see the output corresponding to the system’s default timezone (Central European Time, CET). In the second run, we specify the timezone for New Zealand, which at this time of year is on daylight saving time, 12 hours ahead of CET.

$ ./show_time  
ctime() of time() value is:  Tue Feb  1 10:25:56 2011  
asctime() of local time is:  Tue Feb  1 10:25:56 2011  
strftime() of local time is: Tuesday, 01 Feb 2011, 10:25:56 CET  
$ TZ=":Pacific/Auckland" ./show_time  
ctime() of time() value is:  Tue Feb  1 22:26:19 2011  
asctime() of local time is:  Tue Feb  1 22:26:19 2011  
strftime() of local time is: Tuesday, 01 February 2011, 22:26:19 NZDT

**Listing 10-4:** Demonstrate the effect of timezones and locales

_________________________________________________________ time/show_time.c  
  
#include <time.h>  
#include <locale.h>  
#include "tlpi_hdr.h"  
  
#define BUF_SIZE 200  
  
int  
main(int argc, char *argv[])  
{  
    time_t t;  
    struct tm *loc;  
    char buf[BUF_SIZE];  
  
    if (setlocale(LC_ALL, "") == NULL)  
        errExit("setlocale");   /* Use locale settings in conversions */  
  
    t = time(NULL);  
  
    printf("ctime() of time() value is: %s", ctime(&t));  
  
    loc = localtime(&t);  
    if (loc == NULL)  
        errExit("localtime");  
  
    printf("asctime() of local time is: %s", asctime(loc));  
  
    if (strftime(buf, BUF_SIZE, "%A, %d %B %Y, %H:%M:%S %Z", loc) == 0)  
        fatal("strftime returned 0");  
    printf("strftime() of local time is: %s\n", buf);  
  
    exit(EXIT_SUCCESS);  
}  
_________________________________________________________ time/show_time.c

SUSv3 defines two general ways in which the TZ environment variable can be set. As just described, TZ can be set to a character sequence consisting of a colon plus a string that identifies the timezone in an implementation-specific manner, typically as a pathname for a timezone description file. (Linux and some other UNIX implementations permit the colon to be omitted when using this form, but SUSv3 doesn’t specify this; for portability, we should always include the colon.)

The other method of setting TZ is fully specified in SUSv3. In this method, we assign a string of the following form to TZ:

_std offset_ [ _dst_ [ _offset_ ][ , _start-date_ [ /_time_ ] , _end-date_ [ /_time_ ]]]

Spaces are included in the line above for clarity, but none should appear in the TZ value. The brackets ([]) are used to indicate optional components.

The _std_ and _dst_ components are strings that define names for the standard and DST timezones; for example, _CET_ and _CEST_ for Central European Time and Central European Summer Time. The _offset_ in each case specifies the positive or negative adjustment to add to the local time to convert it to UTC. The final four components provide a rule describing when the change from standard time to DST occurs.

The dates can be specified in a variety of forms, one of which is M_m_._n_._d_. This notation means day _d_ (0 = Sunday, 6 = Saturday) of week _n_ (1 to 5, where 5 always means the last _d_ day) of month _m_ (1 to 12). If the _time_ is omitted, it defaults to 02:00:00 (2 AM) in each case.

Here is how we could define TZ for Central Europe, where standard time is one hour ahead of UTC, and DST—running from the last Sunday in March to the last Sunday in October—is 2 hours ahead of UTC:

TZ="CET-1:00:00CEST-2:00:00,M3.5.0,M10.5.0"

We omitted the specification of the time for the DST changeover, since it occurs at the default of 02:00:00. Of course, the preceding form is less readable than the near equivalent:

TZ=":Europe/Berlin"

### **10.4 Locales**

Several thousand languages are spoken across the world, of which a significant percentage are regularly used on computer systems. Furthermore, different countries use different conventions for displaying information such as numbers, currency amounts, dates, and times. For example, in most European countries, a comma, rather than a decimal point, is used to separate the integer and fractional parts of (real) numbers, and most countries use formats for writing dates that are different from the _MM/DD/YY_ format used in the United States. SUSv3 defines a _locale_ as the “subset of a user’s environment that depends on language and cultural conventions.”

Ideally, all programs designed to run in more than one location should deal with locales in order to display and input information in the user’s preferred language and format. This constitutes the complex subject of _internationalization_. In the ideal world, we would write a program once, and then, wherever it was run, it would automatically do the right things when performing I/O; that is, it would perform the task of _localization_. Internationalizing programs is a somewhat time-consuming job, although various tools are available to ease the task. Program libraries such as _glibc_ also provide facilities to help with internationalization.

The term _internationalization_ is often written as _I18N_, for _I_ plus 18 letters plus _N_. As well as being quicker to write, this term has the advantage of avoiding the difference in the spelling of the term itself in British and American English.

##### **Locale definitions**

Like timezone information, locale information tends to be both voluminous and volatile. For this reason, rather than requiring each program and library to store locale information, the system maintains this information in files in standard formats.

Locale information is maintained in a directory hierarchy under /usr/share/locale (or /usr/lib/locale in some distributions). Each subdirectory under this directory contains information about a particular locale. These directories are named using the following convention:

_language_[__territory_[._codeset_]][@_modifier_]

The _language_ is a two-letter ISO language code, and the _territory_ is a two-letter ISO country code. The _codeset_ designates a character-encoding set. The _modifier_ provides a means of distinguishing multiple locale directories whose language, territory, and codeset are the same. An example of a complete locale directory name is de_DE.utf-8@euro, as the locale for: German language, Germany, UTF-8 character encoding, employing the euro as the monetary unit.

As indicated by the brackets shown in the directory naming format, various parts of the name of a locale directory can be omitted. Often the name consists of just a language and a territory. Thus, the directory en_US is the locale directory for the (English-speaking) United States, and fr_CH is the locale directory for the French-speaking region of Switzerland.

The _CH_ stands for _Confoederatio Helvetica_, the Latin (and thus locally language-neutral) name for Switzerland. With four official national languages, Switzerland is an example of a locale analog of a country with multiple timezones.

When we specify a locale to be used within a program, we are, in effect, specifying the name of one of the subdirectories under /usr/share/locale. If the locale specified to the program doesn’t match a locale directory name exactly, then the C library searches for a match by stripping components from the specified locale in the following order:

1. codeset
    
2. normalized codeset
    
3. territory
    
4. modifier
    

The normalized codeset is a version of the codeset name in which all nonalphanumeric characters are removed, all letters are converted to lowercase, and the resulting string is preprended with the characters iso. The aim of normalizing is to handle variations in the capitalization and punctuation (e.g., extra hyphens) of codeset names.

As an example of this stripping process, if the locale for a program is specified as fr_CH.utf-8, but no locale directory by that name exists, then the fr_CH locale directory will be matched if it exists. If the fr_CH directory doesn’t exist, then the fr locale directory will be used. In the unlikely event that the fr directory doesn’t exist, then the _setlocale()_ function, described shortly, will report an error.

The file /usr/share/locale/locale.alias defines alternative ways of specifying locales to a program. See the _locale.aliases(5)_ manual page for details.

Under each locale subdirectory is a standard set of files that specify the conventions for this locale, as shown in [Table 10-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10table2). Note the following further points concerning the information in this table:

• The LC_COLLATE file defines a set of rules describing how the characters in a character set are ordered (i.e., the “alphabetical” order for the character set). These rules determine the operation of the _strcoll(3)_ and _strxfrm(3)_ functions. Even languages using Latin-based scripts don’t follow the same ordering rules. For example, several European languages have additional letters that, in some cases, sort after the letter _Z_. Other special cases include the Spanish two-letter sequence _ll_, which sorts as a single letter after _l_, and the German umlauted characters such as _ä_, which corresponds to _ae_ and sorts as those two letters.

• The LC_MESSAGES directory is one step toward internationalizing the messages displayed by a program. More extensive internationalization of program messages can be accomplished through the use of either message catalogs (see the _catopen(3)_ and _catgets(3)_ manual pages) or the GNU _gettext_ API (available at _[http://www.gnu.org/](http://www.gnu.org/)_).

Version 2.2.2 of _glibc_ introduced a number of new, nonstandard locale categories. LC_ADDRESS defines rules for the locale-specific representation of a postal address. LC_IDENTIFICATION specifies information identifying the locale. LC_MEASUREMENT defines the measurement system for the locale (e.g., metric versus imperial). LC_NAME defines the locale-specific rules for representation of a person’s names and title. LC_PAPER defines the standard paper size for the locale (e.g., US letter versus the A4 format used in most other countries). LC_TELEPHONE defines the rules for locale-specific representation of domestic and international telephone numbers, as well as the international country prefix and international dial-out prefix.

**Table 10-2:** Contents of locale-specific subdirectories

|**Filename**|**Purpose**|
|---|---|
|LC_CTYPE|A file containing character classifications (see _isalpha(3)_) and rules for case conversion|
|LC_COLLATE|A file containing the collation rules for a character set|
|LC_MONETARY|A file containing formatting rules for monetary values (see _localeconv(3)_ and <locale.h>)|
|LC_NUMERIC|A file containing formatting rules for numbers other than monetary values (see _localeconv(3)_ and <locale.h>)|
|LC_TIME|A file containing formatting rules for dates and times|
|LC_MESSAGES|A directory containing files specifying formats and values used for affirmative and negative (yes/no) responses|

The actual locales that are defined on a system can vary. SUSv3 doesn’t make any requirements about this, except that a standard locale called _POSIX_ (and synonymously, _C_, a name that exists for historical reasons) must be defined. This locale mirrors the historical behavior of UNIX systems. Thus, it is based on an ASCII character set, and uses English for names of days and months, and for yes/no responses. The monetary and numeric components of this locale are undefined.

The _locale_ command displays information about the current locale environment (within the shell). The command _locale –a_ lists the full set of locales defined on the system.

##### **Specifying the locale for a program**

The _setlocale()_ function is used to both set and query a program’s current locale.

#include <locale.h>  
  
char *setlocale(int category, const char *locale);

Returns pointer to a (usually statically allocated) string identifying the new or current locale on success, or NULL on error

The _category_ argument selects which part of the locale to set or query, and is specified as one of a set of constants whose names are the same as the locale categories listed in [Table 10-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10table2). Thus, for example, it is possible to set the locale for time displays to be Germany, while setting the locale for monetary displays to US dollars. Alternatively, and more commonly, we can use the value LC_ALL to specify that we want to set all aspects of the locale.

There are two different methods of setting the locale using _setlocale()_. The _locale_ argument may be a string specifying one of the locales defined on the system (i.e., the name of one of the subdirectories under /usr/lib/locale), such as de_DE or en_US. Alternatively, _locale_ may be specified as an empty string, meaning that locale settings should be taken from environment variables:

setlocale(LC_ALL, "");

We must make this call in order for a program to be cognizant of the locale environment variables. If the call is omitted, these environment variables will have no effect on the program.

When running a program that makes a _setlocale(LC_ALL, “”)_ call, we can control various aspects of the locale using a set of environment variables whose names again correspond to the categories listed in [Table 10-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10table2): LC_CTYPE, LC_COLLATE, LC_MONETARY, LC_NUMERIC, LC_TIME, and LC_MESSAGES. Alternatively, we can use the LC_ALL or the LANG environment variable to specify the setting of the entire locale. If more than one of the preceding variables is set, then LC_ALL has precedence over all of the other LC_* environment variables, and LANG has lowest precedence. Thus, it is possible to use LANG to set a default locale for all categories, and then use individual LC_* variables to set aspects of the locale to something other than this default.

As its result, _setlocale()_ returns a pointer to a (usually statically allocated) string that identifies the locale setting for this category. If we are interested only in discovering the current locale setting, without changing it, then we can specify the _locale_ argument as NULL.

Locale settings control the operation of a wide range of GNU/Linux utilities, as well as many functions in _glibc_. Among these are the functions _strftime()_ and _strptime()_ ([Section 10.2.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev2sec03)), as shown by the results from _strftime()_ when we run the program in [Listing 10-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10ex4) in a number of different locales:

$ LANG=de_DE ./show_time                        German locale  
ctime() of time() value is:  Tue Feb  1 12:23:39 2011  
asctime() of local time is:  Tue Feb  1 12:23:39 2011  
strftime() of local time is: Dienstag, 01 Februar 2011, 12:23:39 CET

The next run demonstrates that the LC_TIME has precedence over LANG:

$ LANG=de_DE LC_TIME=it_IT ./show_time          German and Italian locales  
ctime() of time() value is:  Tue Feb  1 12:24:03 2011  
asctime() of local time is:  Tue Feb  1 12:24:03 2011  
strftime() of local time is: martedì, 01 febbraio 2011, 12:24:03 CET

And this run demonstrates that LC_ALL has precedence over LC_TIME:

$ LC_ALL=fr_FR LC_TIME=en_US ./show_time        French and US locales  
ctime() of time() value is:  Tue Feb  1 12:25:38 2011  
asctime() of local time is:  Tue Feb  1 12:25:38 2011  
strftime() of local time is: mardi, 01 février 2011, 12:25:38 CET

### **10.5 Updating the System Clock**

We now look at two interfaces that update the system clock: _settimeofday()_ and _adjtime()_. These interfaces are rarely used by application programs (since the system time is usually maintained by tools such as the _Network Time Protocol_ daemon), and they require that the caller be privileged (CAP_SYS_TIME).

The _settimeofday()_ system call performs the converse of _gettimeofday()_ (which we described in [Section 10.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec01)): it sets the system’s calendar time to the number of seconds and microseconds specified in the _timeval_ structure pointed to by _tv_.

#define _BSD_SOURCE  
#include <sys/time.h>  
  
int settimeofday(const struct timeval *tv, const struct timezone *tz);

Returns 0 on success, or –1 on error

As with _gettimeofday()_, the use of the _tz_ argument is obsolete, and this argument should always be specified as NULL.

The microsecond precision of the _tv.tv_usec_ field doesn’t mean that we have microsecond accuracy in controlling the system clock, since the clock’s granularity may be larger than one microsecond.

Although _settimeofday()_ is not specified in SUSv3, it is widely available on other UNIX implementations.

Linux also provides the _stime()_ system call for setting the system clock. The difference between _settimeofday()_ and _stime()_ is that the latter call allows the new calendar time to be expressed with a precision of only 1 second. As with _time()_ and _gettimeofday()_, the reason for the existence of both _stime()_ and _settimeofday()_ is historical: the latter, more precise call was added by 4.2BSD.

Abrupt changes in the system time of the sort caused by calls to _settimeofday()_ can have deleterious effects on applications (e.g., _make(1)_, a database system using timestamps, or time-stamped log files) that depend on a monotonically increasing system clock. For this reason, when making small changes to the time (of the order of a few seconds), it is usually preferable to use the _adjtime()_ library function, which causes the system clock to gradually adjust to the desired value.

#define _BSD_SOURCE  
#include <sys/time.h>  
  
int adjtime(struct timeval *delta, struct timeval *olddelta);

Returns 0 on success, or –1 on error

The _delta_ argument points to a _timeval_ structure that specifies the number of seconds and microseconds by which to change the time. If this value is positive, then a small amount of additional time is added to the system clock each second, until the desired amount of time has been added. If the _delta_ value is negative, the clock is slowed down in a similar fashion.

The rate of clock adjustment on Linux/x86-32 amounts to 1 second per 2000 seconds (or 43.2 seconds per day).

It may be that an incomplete clock adjustment was in progress at the time of the _adjtime()_ call. In this case, the amount of remaining, unadjusted time is returned in the _timeval_ structure pointed to by _olddelta_. If we are not interested in this value, we can specify _olddelta_ as NULL. Conversely, if we are interested only in knowing the currently outstanding time correction to be made, and don’t want to change the value, we can specify the _delta_ argument as NULL.

Although not specified in SUSv3, _adjtime()_ is available on most UNIX implementations.

On Linux, _adjtime()_ is implemented on top of a more general (and complex) Linux-specific system call, _adjtimex()_. This system call is employed by the _Network Time Protocol_ (NTP) daemon. For further information, refer to the Linux source code, the Linux _adjtimex(2)_ manual page, and the NTP specification ([[Mills, 1992](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib69)]).

### **10.6 The Software Clock (Jiffies)**

The accuracy of various time-related system calls described in this book is limited to the resolution of the system _software clock_, which measures time in units called _jiffies_. The size of a jiffy is defined by the constant HZ within the kernel source code. This is the unit in which the kernel allocates the CPU to processes under the round-robin time-sharing scheduling algorithm ([Section 35.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch35.xhtml#ch35lev1sec01)).

On Linux/x86-32 in kernel versions up to and including 2.4, the rate of the software clock was 100 hertz; that is, a jiffy is 10 milliseconds.

Because CPU speeds have greatly increased since Linux was first implemented, in kernel 2.6.0, the rate of the software clock was raised to 1000 hertz on Linux/x86-32. The advantages of a higher software clock rate are that timers can operate with greater accuracy and time measurements can be made with greater precision. However, it isn’t desirable to set the clock rate to arbitrarily high values, because each clock interrupt consumes a small amount of CPU time, which is time that the CPU can’t spend executing processes.

Debate among kernel developers eventually resulted in the software clock rate becoming a configurable kernel option (under _Processor type and features, Timer frequency_). Since kernel 2.6.13, the clock rate can be set to 100, 250 (the default), or 1000 hertz, giving jiffy values of 10, 4, and 1 milliseconds, respectively. Since kernel 2.6.20, a further frequency is available: 300 hertz, a number that divides evenly for two common video frame rates: 25 frames per second (PAL) and 30 frames per second (NTSC).

### **10.7 Process Time**

Process time is the amount of CPU time used by a process since it was created. For recording purposes, the kernel separates CPU time into the following two components:

• _User CPU time_ is the amount of time spent executing in user mode. Sometimes referred to as _virtual time_, this is the time that it appears to the program that it has access to the CPU.

• _System CPU time_ is amount of time spent executing in kernel mode. This is the time that the kernel spends executing system calls or performing other tasks on behalf of the program (e.g., servicing page faults).

Sometimes, we refer to process time as the _total CPU time_ consumed by the process.

When we run a program from the shell, we can use the _time(1)_ command to obtain both process time values, as well as the real time required to run the program:

$ time ./myprog  
real    0m4.84s  
user    0m1.030s  
sys     0m3.43s

The _times()_ system call retrieves process time information, returning it in the structure pointed to by _buf_.

#include <sys/times.h>  
  
clock_t times(struct tms *buf);

Returns number of clock ticks (_sysconf(_SC_CLK_TCK)_) since “arbitrary” time in past on success, or _(clock_t)_ –1 on error

This _tms_ structure pointed to by _buf_ has the following form:

struct tms {  
    clock_t tms_utime;   /* User CPU time used by caller */  
    clock_t tms_stime;   /* System CPU time used by caller */  
    clock_t tms_cutime;  /* User CPU time of all (waited for) children */  
    clock_t tms_cstime;  /* System CPU time of all (waited for) children */  
};

The first two fields of the _tms_ structure return the user and system components of CPU time used so far by the calling process. The last two fields return information about the CPU time used by all child processes that have terminated and for which the parent (i.e., the caller of _times()_) has done a _wait()_ system call.

The _clock_t_ data type used to type the four fields of the _tms_ structure is an integer type that measures time in units called _clock ticks_. We can call _sysconf(_SC_CLK_TCK)_ to obtain the number of clock ticks per second, and then divide a _clock_t_ value by this number to convert to seconds. (We describe _sysconf()_ in [Section 11.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch11.xhtml#ch11lev1sec02).)

On most Linux hardware architectures, _sysconf(_SC_CLK_TCK)_ returns the number 100. This corresponds to the kernel constant USER_HZ. However, USER_HZ can be defined with a value other than 100 on a few architectures, such as Alpha and IA-64.

On success, _times()_ returns the elapsed (real) time in clock ticks since some arbitrary point in the past. SUSv3 deliberately does not specify what this point is, merely stating that it will be constant during the life of the calling process. Therefore, the only portable use of this return value is to measure elapsed time in the execution of the process by calculating the difference in the value returned by pairs of _times()_ calls. However, even for this use, the return value of _times()_ is unreliable, since it can overflow the range of _clock_t_, at which point the value would cycle to start again at 0 (i.e., a later _times()_ call could return a number that is lower than an earlier _times()_ call). The reliable way to measure the passage of elapsed time is to use _gettimeofday()_ (described in [Section 10.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec01)).

On Linux, we can specify _buf_ as NULL; in this case, _times()_ simply returns a function result. However, this is not portable. The use of NULL for _buf_ is not specified in SUSv3, and many other UNIX implementations require a non-NULL value for this argument.

The _clock()_ function provides a simpler interface for retrieving the process time. It returns a single value that measures the total (i.e., user plus system) CPU time used by the calling process.

#include <time.h>  
  
clock_t clock(void);

Returns total CPU time used by calling process measured in CLOCKS_PER_SEC, or _(clock_t)_ –1 on error

The value returned by _clock()_ is measured in units of CLOCKS_PER_SEC, so we must divide by this value to arrive at the number of seconds of CPU time used by the process. CLOCKS_PER_SEC is fixed at 1 million by POSIX.1, regardless of the resolution of the underlying software clock ([Section 10.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec06)). The accuracy of _clock()_ is nevertheless limited to the resolution of the software clock.

Although the _clock_t_ return type of _clock()_ is the same data type that is used in the _times()_ call, the units of measurement employed by these two interfaces are different. This is the result of historically conflicting definitions of _clock_t_ in POSIX.1 and the C programming language standard.

Even though CLOCKS_PER_SEC is fixed at 1 million, SUSv3 notes that this constant could be an integer variable on non-XSI-conformant systems, so that we can’t portably treat it as a compile-time constant (i.e., we can’t use it in #ifdef preprocessor expressions). Because it may be defined as a long integer (i.e., 1000000L), we always cast this constant to _long_ so that we can portably print it with _printf()_ (see [Section 3.6.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch03.xhtml#ch03lev2sec04)).

SUSv3 states that _clock()_ should return “the processor time used by the process.” This is open to different interpretations. On some UNIX implementations, the time returned by _clock()_ includes the CPU time used by all waited-for children. On Linux, it does not.

##### **Example program**

The program in [Listing 10-5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10ex5) demonstrates the use of the functions described in this section. The _displayProcessTimes()_ function prints the message supplied by the caller, and then uses _clock()_ and _times()_ to retrieve and display process times. The main program makes an initial call to _displayProcessTimes()_, and then executes a loop that consumes some CPU time by repeatedly calling _getppid()_, before again calling _displayProcessTimes()_ once more to see how much CPU time has been consumed within the loop. When we use this program to call _getppid()_ 10 million times, this is what we see:

$ ./process_time 10000000  
CLOCKS_PER_SEC=1000000 sysconf(_SC_CLK_TCK)=100  
  
At program start:  
        clock() returns: 0 clocks-per-sec (0.00 secs)  
        times() yields: user CPU=0.00; system CPU: 0.00  
After getppid() loop:  
        clock() returns: 2960000 clocks-per-sec (2.96 secs)  
        times() yields: user CPU=1.09; system CPU: 1.87

**Listing 10-5:** Retrieving process CPU times

_______________________________________________________ time/process_time.c  
  
#include <sys/times.h>  
#include <time.h>  
#include "tlpi_hdr.h"  
  
static void             /* Display 'msg' and process times */  
displayProcessTimes(const char *msg)  
{  
    struct tms t;  
    clock_t clockTime;  
    static long clockTicks = 0;  
  
    if (msg != NULL)  
        printf("%s", msg);  
    if (clockTicks == 0) {      /* Fetch clock ticks on first call */  
        clockTicks = sysconf(_SC_CLK_TCK);  
        if (clockTicks == -1)  
            errExit("sysconf");  
    }  
  
    clockTime = clock();  
    if (clockTime == -1)  
        errExit("clock");  
  
    printf("        clock() returns: %ld clocks-per-sec (%.2f secs)\n",  
            (long) clockTime, (double) clockTime / CLOCKS_PER_SEC);  
  
    if (times(&t) == -1)  
        errExit("times");  
    printf("        times() yields: user CPU=%.2f; system CPU: %.2f\n",  
            (double) t.tms_utime / clockTicks,  
            (double) t.tms_stime / clockTicks);  
}  
  
int  
main(int argc, char *argv[])  
{  
    int numCalls, j;  
  
    printf("CLOCKS_PER_SEC=%ld sysconf(_SC_CLK_TCK)=%ld\n\n",  
            (long) CLOCKS_PER_SEC, sysconf(_SC_CLK_TCK));  
  
    displayProcessTimes("At program start:\n");  
  
    numCalls = (argc > 1) ? getInt(argv[1], GN_GT_0, "num-calls") : 100000000;  
    for (j = 0; j < numCalls; j++)  
        (void) getppid();  
  
    displayProcessTimes("After getppid() loop:\n");  
  
    exit(EXIT_SUCCESS);  
}  
_______________________________________________________ time/process_time.c

### **10.8 Summary**

Real time corresponds to the everyday definition of time. When real time is measured from some standard point, we refer to it as calendar time, by contrast with elapsed time, which is measured from some point (usually the start) in the life of a process.

Process time is the amount of CPU time used by a process, and is divided into user and system components.

Various system calls enable us to get and set the system clock value (i.e., calendar time, as measured in seconds since the Epoch), and a range of library functions allow conversions between calendar time and other time formats, including broken-down time and human-readable character strings. Describing such conversions took us into a discussion of locales and internationalization.

Using and displaying times and dates is an important part of many applications, and we’ll make frequent use of the functions described in this chapter in later parts of this book. We also say a little more about the measurement of time in [Chapter 23](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch23.xhtml#ch23).

##### **Further information**

Details of how the Linux kernel measures time can be found in [[Love, 2010](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib59)].

An extensive discussion of timezones and internationalization can be found in the GNU C library manual (online at _[http://www.gnu.org/](http://www.gnu.org/)_). The SUSv3 documents also cover locales in detail.

### **10.9 Exercise**

**10-1.**   Assume a system where the value returned by the call _sysconf(_SC_CLK_TCK)_ is 100. Assuming that the _clock_t_ value returned by _times()_ is a signed 32-bit integer, how long will it take before this value cycles so that it restarts at 0? Perform the same calculation for the CLOCKS_PER_SEC value returned by _clock()_.
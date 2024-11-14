To display current time, enter:

`$ date`

Sample outputs:
`Wed Oct 27 16:50:41 IST 2010`

You can only display time, enter:  
`$ date +"%T"`  
Sample outputs:
`16:51:05`

The following command use your local standards to display date and time:  
`$ date +"%c"`  
Sample outputs:

Wednesday 27 October 2010 04:52:06 PM IST

To display the date and time in a specified format, enter:  
`date +"%r %a %d %h %y (Julian Date: %j)"`  
Sample outputs:
`12:52:58 AM Sun 20 Sep 15 (Julian Date: 263)`

### A list of date command field descriptors

       %%     a literal %
 
       %a     locale's abbreviated weekday name (e.g., Sun)
 
       %A     locale's full weekday name (e.g., Sunday)
 
       %b     locale's abbreviated month name (e.g., Jan)
 
       %B     locale's full month name (e.g., January)
 
       %c     locale's date and time (e.g., Thu Mar  3 23:05:25 2005)
 
       %C     century; like %Y, except omit last two digits (e.g., 20)
 
       %d     day of month (e.g., 01)
 
       %D     date; same as %m/%d/%y
 
       %e     day of month, space padded; same as %_d
 
       %F     full date; same as %Y-%m-%d
 
       %g     last two digits of year of ISO week number (see %G)
 
       %G     year of ISO week number (see %V); normally useful only with %V
 
       %h     same as %b
 
       %H     hour (00..23)
 
       %I     hour (01..12)
 
       %j     day of year (001..366)
 
       %k     hour, space padded ( 0..23); same as %_H
 
       %l     hour, space padded ( 1..12); same as %_I
 
       %m     month (01..12)
 
       %M     minute (00..59)
 
       %n     a newline
 
       %N     nanoseconds (000000000..999999999)
       %p     locale's equivalent of either AM or PM; blank if not known
 
       %P     like %p, but lower case
 
       %r     locale's 12-hour clock time (e.g., 11:11:04 PM)
 
       %R     24-hour hour and minute; same as %H:%M
 
       %s     seconds since 1970-01-01 00:00:00 UTC
 
       %S     second (00..60)
 
       %t     a tab
 
       %T     time; same as %H:%M:%S
 
       %u     day of week (1..7); 1 is Monday
 
       %U     week number of year, with Sunday as first day of week (00..53)
 
       %V     ISO week number, with Monday as first day of week (01..53)
 
       %w     day of week (0..6); 0 is Sunday
 
       %W     week number of year, with Monday as first day of week (00..53)
 
       %x     locale's date representation (e.g., 12/31/99)
 
       %X     locale's time representation (e.g., 23:13:48)
 
       %y     last two digits of year (00..99)
 
       %Y     year
 
       %z     +hhmm numeric time zone (e.g., -0400)
 
       %:z    +hh:mm numeric time zone (e.g., -04:00)
 
       %::z   +hh:mm:ss numeric time zone (e.g., -04:00:00)
 
       %:::z  numeric time zone with :  to  necessary  precision  (e.g.,  -04,
              +05:30)
 
       %Z     alphabetic time zone abbreviation (e.g., EDT)
       By  default,  date  pads  numeric  fields  with  zeroes.  The following
       optional flags may follow '%':
 
       -      (hyphen) do not pad the field
 
       _      (underscore) pad with spaces
 
       0      (zero) pad with zeros
 
       ^      use upper case if possible
 
       #      use opposite case if possible


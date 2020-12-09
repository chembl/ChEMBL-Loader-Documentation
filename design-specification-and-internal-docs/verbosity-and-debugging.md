# Verbosity and debugging

Verbosity levels higher than 9 are reserved for calls which output particular data structures, or related information on the content flowing through the program. These can be very useful during debugging. The levels assigned to such information progrssively inclrease throughout the program. 

For example:

* Levels up to 1000 are used for 'mode 1' operations. 
* Levels 1000-2000 are reserved for 'mode 2' operations

These data be manually outputted by raising '-v' to the level required, but this will also output all levels from 0 upto this raised level. 

Therefore, a more sensible way of viewing such structure is to use the '-w' or '-j' option. These two options have identical effects. 

* Setting one of these to a particular level, and not setting the other, will output all calls with levels equal to this option. 
* Setting both -w and -j to different levels will output all verb calls inbetween \(and including\) these two level values. This can be useful if a number of data structures are required to be viewed at once.
* When both are set to the SAME value, all calls with levels equal to or higher than this value will be outputted.  
  * The value of the data structure or related info is not shown, but instead is subsituted for the 'level' number. 
  * In addition, information on the calling subroutine of the verb function, and the data Type of the data structure is shown. 
  * This 'special case' is useful for identifying 'level' values that may be useful for a particular debugging process. 
  * In this special case, all '-v' levels are also outputted, but no 'substitution' occurs for those.

Here's some examples of how this works...

| option settings | 'verb' will output calls with these 'levels' \(inclusive\) |
| :--- | :--- |
| -v 6 -w 29 | 0-6 and 29 |
| -v 6 -j 29 | 0-6 and 29 |
| -v 3 -w 29 -j 1200 | 0-3, and 29-1200 |
| -v 3 -j 29 -w 1200 | 0-3, and 29-1200 |
| -v 2 | 0-2 0 |
| -j 29 -w 29 | &gt;= 29 \(but no 'values'\) |

Another useful option used in debugging is the option '-b'. Using this will append 'level' and timestamp information to the single record output of verb.


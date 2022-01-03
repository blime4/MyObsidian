The functions vprintf(), vfprintf(), vdprintf(), vsprintf(), vsnprintf() are equivalent to the functions printf(),fprintf(), dprintf(), sprintf(), snprintf(), respectively, 

except that they are called with a va_list instead of a variable number of arguments.  

These functions do not call the va_end  macro.   Because  they  invoke  the  va_arg
macro, the value of ap is undefined after the call.
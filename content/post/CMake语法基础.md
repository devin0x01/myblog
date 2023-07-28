
# 基本表达式
[if — CMake 3\.27\.1 Documentation](https://cmake.org/cmake/help/latest/command/if.html)  
[Cmake中的条件判断if/elseif/else \- 简书](https://www.jianshu.com/p/3958cff0318b)

## if(\<constant\>)
>True if the constant is 1, ON, YES, TRUE, Y, or a non-zero number (including floating point numbers). False if the constant is 0, OFF, NO, FALSE, N, IGNORE, NOTFOUND, the empty string, or ends in the suffix -NOTFOUND. Named boolean constants are case-insensitive. If the argument is not one of these specific constants, it is treated as a variable or string (see Variable Expansion further below) and one of the following two forms applies.

## if(\<variable\>)
>True if given a variable that is defined to a value that is not a false constant. False otherwise, including if the variable is undefined. Note that macro arguments are not variables. Environment Variables also cannot be tested this way, e.g. if(ENV{some_var}) will always evaluate to false.
JS library ideas

## Better array utils

Basically the same old array utils that you can use with native JS, but optimized. For example, without this library, when you call 
array.reduce().filter().map(), it loops through the array 3 times for each array function. The idea with the library is to use the Builder pattern to build out the process before looping through the array, allowing you to loop through it only once, resulting in better performance.
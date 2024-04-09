PHP is a loosely typed language. That means developers don’t need to declare the variable types when defining the variables. This means that PHP will automatically assign the variable type during the execution of the program. This is really helpful in situations where the input type is dynamic. But, the issue with this concept is that, if improperly implemented, can lead to security issues.

Since PHP is a loosely typed language, PHP will convert the variables into a comparable type first, when there arises a need to compare two variables. This behavior of PHP is called as Type Juggling.

Suppose the target has setup the following PHP code to authenticate using a hardcoded hash.

![](https://secnigma.files.wordpress.com/2021/04/image-68.png?w=677)

The `strcmp` function will return the following output on different situations. [Source.](https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf)

![](https://secnigma.files.wordpress.com/2021/04/image-71.png?w=423)

So, When both variables are equal, `strcmp` will return `0`.

From the client side, we can control two things. The POST variable name and the data passed through the POST variable. If we change the POST data from `pass="password"` to `pass[]=` , that initiates the Type juggling and PHP will convert the POST variable we passed into an empty array, because of the square brackets.

This breaks the `strcmp` function and `strcmp` will return `0`, bypassing the authentication altogether!
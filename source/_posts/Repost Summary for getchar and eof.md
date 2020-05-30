---
title: 'Repost: Summary for getchar() and eof'
id: 22
categories:
  - c&cpp
date: 2020-05-28 09:24:53

---

*original post date: July 22, 2016*

<!--more-->

Facing classic books, carefully read every word, and then understand. I was looking at *Section 1.5: Character Input and Output* of K&R's *The C Programming Language Second Edition* , and was puzzled by `getchar()` and `EOF`. This may be due to a lack of understanding of how `getchar()` and `EOF` works. Therefore, it seems necessary to summarize, otherwise, a lot of trivial knowledge will fade away after a long time, only write down is the best way.

In fact, the most typical program is just a few lines of code. My Environment Is Debian GNU / Linux, as it is on other systems.

## Two Points for getchar()

### getchar is processing in units of rows

If the first character entered is a valid character, the input is the end-of-file Character EOF (Ctrl + Z for Windows; Ctrl + D for Unix / Linux). Then only if the last input character is a newline character `\n` or a end-of-file character, EOF will be discussed later, will getchar stop, and the whole program will run down. For example, the following sequence:

{% codeblock lang:c %}
while((c = getchar()) != EOF){
    putchar(c);
{% endcodeblock %}
Execute the program, Type: `ABC`, then enter. Then the program executes `putchar(c)` and prints out `ABC`, which, don't forget, also has a carriage return. You can then proceed with the input, and when the newline character is encountered again, the program outputs the input character for that line to the terminal.


For `getchar()`, I'm sure many beginners will ask, isn't `getchar()` read by characters? So, now that I've entered the first character `a`, I must have a while loop `(c = getchar()) != EOF` condition, then `putchar(c)` should be executed to output a character `a` at the terminal. Yes, that's what I always thought when I used `getchar()`, but instead of doing that, the program had to read a newline or end-of-file EOF to get an output.

One explanation for this problem is that, when the creator wrote C, there was no such thing as terminal input. All input was actually read from a file, which was usually in behavioral units. Therefore, only when a newline character is encountered does the program think the input is finished and then take action to execute the rest of the program. Also, the input is accessed as a File, so ending a File input requires EOF. This is why `getchar()` ends the input exit with EOF.


### The return value of getchar() is typically a character, but it can also be negative, which is EOF.

The point to emphasize here is that the `getchar()` function usually returns the characters entered by the terminal, which correspond to non-negative ASCII values in the character system. So, a lot of times, we would write code like this:

{% codeblock lang:c %}
char c;
c = getchar();
{% endcodeblock %}

There's a good chance something could go wrong. Because the `getchar()` function returns EOF when it encounters Ctrl+D on Linux, the EOF, in addition to the characters entered by the terminal, which is generally defined as `-1` in the library. Therefore, in this case, the `getchar()` function returns a negative value, and it is incorrect to assign a negative value to a variable of type `char`. In order to be able to define a variable that contains all the possible values returned by the `getchar()` function, the proper way to define it is as follows, specifically mentioned in K&R:

{% codeblock lang:c %}
int c;
c = getchar();
{% endcodeblock %}

## Two Points for EOF

### EOF As the end-of-file condition

EOF is the end-of-file character, but it is not always possible to end a file by typing Ctrl+D in Windows:

1. when the `getchar()` function is executed, to input the first character directly input Ctrl+D, You can jump out of `getchar()` to execute other parts of the program;

2. type Ctrl+D when the character you entered earlier is a newline character;

3. Enter Ctrl+D twice in a row if there is a character input before and it is not a newline character. The second input Ctrl+D functions as the end of the file. The first input Ctrl+D functions as shown below.

In fact, all three cases can be summed up by saying that Ctrl+D is the end of the file only if getchar prompts for a new input.

### When EOF is the line Terminator, typing Ctrl+D does not end Getchar, but only causes getchar() to prompt for the next round of input

This is mainly when a new line of `getchar()` input, when some characters can not contain a newline, directly press Ctrl+D, at this point the Ctrl+D is not the end of the file, but only the function equivalent to the newline, ends the current input. In the example above, if you type `abc` at run time and then Ctrl+D, The output is:

{% codeblock %}
abcabc
{% endcodeblock %}

Note: The first set of `abc` is entered from the terminal, and then Ctrl+D is entered to output the second set of `abc`, with the cursor pausing after the C of the second set of characters, and then a new input can be made. At this point, if you type Ctrl+D again, it acts as the end of the file, ending `getchar()`.

If you type `abc`, then press Enter to enter a newline character, the terminal appears as:

{% codeblock %}
abc       // the first line with Enter \r
abc       // the second line
          // the third line
{% endcodeblock %}

Where the first line is the terminal input, the second line is the terminal output, and the cursor stops at the third line, waiting for a new terminal input.

You can also see the difference in the output when Ctrl+D and newline are used as line endings, respectively.

The function of EOF can also be summarized as follows: When the terminal has character input, Ctrl+D produces EOF equivalent to the end of the line input, will cause a new round of `getchar()` input; when the terminal has no character input or can say when `getchar()` reads a new round of input, type Ctrl+D and the resulting EOF corresponds to the end of the file, ending the execution of `getchar()`.

The summary of EOF in the second part of this article applies to the terminal driver in a one-line-at-a-time mode. That is, while `getchar()` and `putchar()` do follow one character at a time. But the terminal driver is in a one-line-at-a-time mode, where the input only ends at `\n` or Eof, so the output on the terminal is line-by-line.

If you want the terminal to read a character on the end of the input, the following procedure is one way to achieve reference to *C Expert Programming*, with a slight change:

{% codeblock lang:c %}
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    int c;
    /* one-line-at-a-time */
    system("stty codeblock");

    /* one-character-at-a-time */
     c = getchar();
    putchar();

    /* back to one-line-at-a-time */
     system("stty cooked");

    return 0;
}
{% endcodeblock %}

Compile to run the program, when such as enter a character, directly from a character, and then the end of the program.

Thus, due to the different terminal-driven mode, the conditions for the end of the Getchar input are different. A carriage return or EOF is required in normal mode, but in one character at a time mode, it ends after one character is entered.

* * *

Origin(invalid link): http://blog.chinaunix.net/u1/53811/showart_421385.html

Repost: http://blog.csdn.net/ithomer/article/details/5669762

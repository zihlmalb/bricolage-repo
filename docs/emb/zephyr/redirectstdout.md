# Redirect stdout to a any output

In many board configurations stdout will be directed to a uart. Usually, when using newlib as libc implementation, this can be achieved by overwriting _write-function.

Since zephyr works with *minimal libc* in its default configuration this approach doesn't work.

Instead *minimal libc* offers the possibility to register a putchar-hook for stdout using the function ``__stdout_hook_install``. All you have to do in order to make this work is following:

- include zephyr/sys/libc-hooks.h
- implement your own putchar function (``int(*)(int c)``)
- register this function with ``__stdout_hook_install``

```
#include <zephyr/sys/libc-hooks.h>

static int myputchar(int c){
	// do whatever you want with the character
	return c;
}


void init_function(void){
	__stdout_hook_install(myputchar);
}
```
Then uncnofigure the console output and the console uart driver in your project configuration *prj.conf*

```
CONFIG_CONSOLE=n
CONFIG_UART_CONSOLE=n
```

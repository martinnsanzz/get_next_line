<h3 align="center">
  <em>Reading a line from a file descriptor is far too tedious.</em>
</h3>


## ⚠️ Disclaimer

- **Full Portfolio:** This repository focuses on this specific project. You can find my entire
42 curriculum 👉 [here](https://github.com/martinnsanzz/42-Curriculum).
- **Subject Rules:** I strictly follow the rules regarding 42 subjects; I cannot share the PDFs,
but I explain the concepts in this README.
- **Archive State:** The code is preserved exactly as it was during evaluation (graded state).
I do not update it, so you can see my progress and mistakes from that time.
- **Academic Integrity:** I encourage you to try the project yourself first. Use this repo only
as a reference, not for copy-pasting. Be patient, you will succeed.

## 📂 Description

The goal of this project is to create a function called **get_next_line** that every time it
gets called reads a line ending with a newline character (`\n`) from a file descriptor
regardless of its size.

The main mechanism of this project is the use of **static variables** to preserve the data
of what was read between calls, specially buffered data that was read past the
previous `\n` but wasn't yet returned. The challenge is to "remember" this extra data for the
next function call without using global variables. It's also teaches you the wonderful world of
leaks and segfault 🏊‍♂️.

The mandatory part only handles a single file descriptor via one static `char *` buffer
while the bonus part handles multiple fd's at the same time (1024 simultaneously, which is
the default soft limit for open fd's in Linux).


## 🔷 Usage

### Clone repository
To use this project you first need to clone the repository in your directory.

```bash
git clone git@github.com:martinnsanzz/get_next_line.git
```

### How to use the function
First you need to include the header on top of your file. Use either based on requirements, to
handle multiple **File descriptors** use the bonus.

```c
#include "get_next_line.h"
#include "get_next_line_bonus.h"
```

To use this function you need to compile the <`.c`> files with your main. Don't forget to define
the **BUFFER_SIZE** (default is 4096/4KB typical minimum allocation unit).

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 your_main.c get_next_line.c get_next_line_utils.c
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 youre_main.c get_next_line_bonus.c get_next_line_utils_bonus.c
```

#### Example Usage

```c
#include "get_next_line.h"
#include <stdio.h>
#include <fcntl.h>

int main(void)
{
    int   fd;
    char  *line;

    fd = open("file.txt", O_RDONLY);
    while (1)
    {
        line = get_next_line(fd);
        if (line == NULL)
            break ;
        printf("%s", line);
        free(line);
    }
    close(fd);
    return (0);
}
```

The caller is responsible for freeing each returned line. `get_next_line` also works on `fd = 0` (standard input).

## 🤖 Algorithm

Two common approaches exist for GNL: a **linked list** (storing chunks as nodes) or a
**single dynamic string** (joining chunks into one buffer). This implementation uses the
string approach because string manipulation with `ft_strjoin` is simpler to reason about
than managing node allocation, traversal, and freeing. It also makes error handling more
straightforward — a `NULL` return from any string operation is easy to catch and propagate.

Each call to `get_next_line` follows these four steps:

> On first call `buffer = NULL`, so it is allocated to an empty string.

1. **`fill_buffer`** — reads from `fd` in chunks of `BUFFER_SIZE` bytes, appending each chunk to the static buffer via `ft_strjoin`, until a `\n` is found or EOF is reached.
2. **`extract_line`** — scans the buffer for `\n` and allocates a new string containing exactly one line (up to and including `\n`, or the full buffer if no `\n` exists).
3. **`trim_buffer`** — allocates a new string containing everything after the first `\n`, discarding the extracted line while preserving any leftover data for the next call.
4. The old buffer pointer is freed after trimming; the new trimmed buffer becomes the new static state.

## 📖 Resources

For the creation of this project the following resources were used:
- [Linux manual page - read()](https://man7.org/linux/man-pages/man2/read.2.html) — Official page for `read()`, used to understand what the function was and how to implement it.
- [GeeksforGeeks - Static Variables in C](https://www.geeksforgeeks.org/c/static-variables-in-c/) — Step by step explanation on how to use static variables and what they are.
- [Code Quoi - Handling a File by its Descriptor in C](https://www.codequoi.com/en/handling-a-file-by-its-descriptor-in-c/) — Used to understand what `fd` was and to test the function with custom `.txt` files. Helpful for the bonus part to handle multiple fds.

**AI usage:**

Claude by Anthropic was used in this project for the following:
- **As a teacher:** To understand complex concepts such as freeing memory correctly, dereferencing pointers, passing a pointer to a pointer, static variables, bytes, and `read()`.
- **Rewriting comments:** In my personal GitHub, all functions have a block comment explaining what it does, parameters, return values, behaviour, and extra notes. These block headers were removed for the submission.
- **Testing:** I wrote several `.txt` files to find edge cases where `get_next_line` wouldn't work as intended.

AI was NOT used to write or generate any code. All function bodies were written by myself.
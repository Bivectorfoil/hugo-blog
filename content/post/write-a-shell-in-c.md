---
title: "[译］教程 - 用C实现一个Shell"
date: 2018-03-31T21:52:55+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["shell", "C"]
categories: ["技术翻译"]
CreativeCommons: true
draft: false
---

> 原文链接：[Tutorial - Write a Shell in C](https://brennan.io/2015/01/16/write-a-shell-in-c/)，作者：[Stepthen Brennan](https://brennan.io/) 16 January 2015



我们很容易自视为”不是一个真正的程序员“。存在那些人尽皆知的程序，使得其中相关的研发者很容易地被推上神坛。尽管研发大型软件项目并不容易，但很多时候这些软件的基本思想是十分简单的。亲自去实现是学习如何成为一个真正的程序员的有趣方法。本文就是关于我如何用C实现简易的Unix Shell的演练，希望其他人也能从中体会到同样的乐趣。

本文所描述的Shell名为：lsh, 可以在[Github](https://github.com/brenns10/lsh)上找到它。

**大学生注意告示！**很多课程会要求你们实现一个Shell，一些教师也知道本文及其代码。如果你是这些课程的其中一员，你不应未经允许地复制这些代码（或复制后修改）。即使如此，我强烈建议不要过度依赖本教程。（关于本段，参考作者的另一篇[博文](https://brennan.io/2016/03/29/dishonesty/)，译者注）



## Shell的基本生命周期



让我们宏观地了解下Shell。一个Shell在它的生命周期中主要做三件事情。

- 初始化：在这一步，Shell会读取并执行自身的配置文件。这些配置文件改变了Shell的操作特性。
- 解释：接下来，Shell从标准输入读取命令（可以是交互命令，或文件）并执行之。
- 终止：在其命令执行后，Shell执行任意关机命令，释放内存并终止。

上述流程是如此普遍以至于可以应用到很多程序中去，我们将在自己的Shell中实现这些流程。我们的Shell是十分简单的，以至于它不需要任何的配置文件，也不会有任何的关机命令。所以，我们只需要调用循环函数然后终止就好。但在架构方面，需要时刻注意程序的生命周期远不止循环。

```c
int main(int argc, char **argv)
{
  // Load config files, if any.
  
  // Run command loop.
  lsh_loop();
  
  // Perform any shutdown/cleanup.
  
  return EXIT_SUCCESS;
}
```

在这你可以看到我刚刚用到的函数，```lsh_loop()```，将会开始循环，解释命令。接下来我们将看到其具体实现。



## Shell的基本循环



我们已经知道到程序应该如何启动了。现在，就程序基本逻辑来说：shell在循环中该怎么做？简单来说，处理命令有三个步骤：

- 读取：从标准输入中读取命令。
- 解析：将命令字符串分隔为程序名和参数。
- 执行：运行解析后的命令。

将上述步骤写成```lsh_loop()```代码:

```c
void lsh_loop(void)
{
  char *line;
  char **args;
  int status;
  
  do {
    printf("> ");
    line = lsh_read_line();
    args = lsh_split_line(line);
    status = lsh_execute(args);
    
    free(line);
    free(args);
  } while (status);
}
```

让我们过一遍代码。前几行是声明。do-while循环更适合用于检查```status```，因为它在检查前会执行一次命令。在本循环中，我们打印一个命令提示符，调用函数读取输入，调用函数将输入分割进```args```， 并执行```args```。最后释放原先创建的参数空间。注意，我们使用由```lsh_execute()```返回的变量```status```来决定何时退出程序。



## 读取输入



读取标准输入听上去似乎很简单，但在C中有点麻烦。棘手的是，你无法提前知道用户会在Shell中输入多少内容。你不能只是简单地分配一块空间，然后希望用户不要超量使用它。因此，你需要从一块空间开始，一旦用户使用量超过了它，立马重新分配更多的空间。这在C中是一个常用策略，我们依据这个想法来实现```lsh_read_line()```。

```c
#define LSH_RL_BUFSIZE 1024
char *lsh_read_line(void)
{
  int bufsize = LSH_RL_BUFSIZE;
  int position = 0;
  char *buffer = malloc(sizeof(char) * bufsize);
  int c;

  if (!buffer) {
    fprintf(stderr, "lsh: allocation error\n");
    exit(EXIT_FAILURE);
  }

  while (1) {
    // Read a character
    c = getchar();

    // If we hit EOF, replace it with a null character and return.
    if (c == EOF || c == '\n') {
      buffer[position] = '\0';
      return buffer;
    } else {
      buffer[position] = c;
    }
    position++;

    // If we have exceeded the buffer, reallocate.
    if (position >= bufsize) {
      bufsize += LSH_RL_BUFSIZE;
      buffer = realloc(buffer, bufsize);
      if (!buffer) {
        fprintf(stderr, "lsh: allocation error\n");
        exit(EXIT_FAILURE);
      }
    }
  }
  
}
```

第一部分是一堆声明。如果你没注意到的话，我建议在接下来的代码中保留旧版C标准风格声明变量的方式。函数主体是一个（无限循环的）```while(1)```循环。在循环中，我们每次读取一个字符（并将其存储为```int```，而不是```char```， 这非常重要！EOF是整数型，而不是字符型，如果你想要将读取的字符与EOF做对比检查，你应该使用```int```。这是C新手常犯的一个错误。）。如果它是换行符，或EOF，我们终止当前字符串并返回它。不然，就将其添加到已存在的字符串中。

接下来，我们检查下一个输入字符是否超出了当前的缓冲区。如果是，我们在继续之前重新分配缓冲区（检查分配错误）。整个流程就是如此了。

对熟悉新版C标准库的人来说，可能会注意到在```stdio.h```头文件中有一个函数```getline()```，它可以提供大部分我们才刚刚实现的功能。坦诚来说，我在完成这份代码之前并不知道有这个函数的存在。这个GUN拓展函数在2008年时被添加到C标准库规范之中，所以大多数的现代Unix系统应该都已配备了。我将保留原来的代码实现，并且鼓励你们应首先学习这样的实现而不是一开始就使用```getline```。若不如此的话，你就是剥夺了自己一次学习的机会了。不管怎样，使用``getline``的话，```lsh_read_line()```函数可改写为如下：

```c
char *lsh_read_line(void)
{
  char *line = NULL;
  ssize_t bufsize = 0; // have getline allocate a buffer for us
  getline(&line, &bufsize, stdin);
  return line;
}
```



## 解析输入

  

好了，如果我们回顾一下loop循环，可以看到我们已经实现了```lsh_read_line()```，也有了输入流。现在，我们需要将输入解析为参数列表。在这里我要做一个简化，在我们的命令行参数中并不支持引号或反斜杠。作为替代，我们使用空格来作为参数间的分割符。所以形如命令 ：```echo "this message"```并不会直接将参数```this message```给传递到```echo```中，而是分为两个参数：```"this"```和```"message"```。

在这样的简化下，我们所需要做的就是使用空格作为分隔符来”解析标记“输入字符串。如此一来我们就可以利用库函数```strtok```来完成一些繁杂的工作。

```c
#define LSH_TOK_BUFSIZE 64
#define LSH_TOK_DELIM " \t\r\n\a"
char **lsh_split_line(char *line)
{
  int bufsize = LSH_TOK_BUFSIZE, position = 0;
  char **tokens = malloc(bufsize * sizeof(char*));
  char *token;

  if (!tokens) {
    fprintf(stderr, "lsh: allocation error\n");
    exit(EXIT_FAILURE);
  }

  token = strtok(line, LSH_TOK_DELIM);
  while (token != NULL) {
    tokens[position] = token;
    position++;

    if (position >= bufsize) {
      bufsize += LSH_TOK_BUFSIZE;
      tokens = realloc(tokens, bufsize * sizeof(char*));
      if (!tokens) {
        fprintf(stderr, "lsh: allocation error\n");
        exit(EXIT_FAILURE);
      }
    }

    token = strtok(NULL, LSH_TOK_DELIM);
  }
  tokens[position] = NULL;
  return tokens;
}
```

如果你觉得这段代码看起来和```lsh_read_line()```十分相似的话，那是因为正是如此！我们采取了同样的策略，使用并动态拓展缓冲区。但这一次，我们用一个以null结尾的指针数组来代替以null结尾的字符数组。

在函数开头，我们调用```strtok```来进行解析```token```。它返回一个指向第一个```token```的指针。```strtok()```实际上做的是返回指向你提供给它的字符串的指针，并在每一个```token```的后面添加```\0```字符。我们将每个指针存储在字符指针数组（缓冲区）中。

最后，如果有必要的话，我们重新分配指针数组。这个过程反复进行直到```strtok```函数不再返回```token```，此时我们终止```tokens```列表。

好了，一旦准备周全后，我们有了一个```tokens```数组，准备开始执行了。那么问题来了，我们应该怎么做？



## Shell如何启动进程



目前，我们开始深入到了Shell工作原理的核心。开启进程是Shell的主要功能。所以编写一个Shell意味着你必须清楚地知道进程中发生了什么，以及它们是如何启动的。正因此，我要稍稍岔开话题，单独地来谈谈Unix中的进程。

在Unix中启动进程只有两种方法。第一种（基本可以不考虑）是通过初始化。可以想见，当Unix系统启动时，它的内核开始加载。一旦其加载和初始化完成，内核只会开始运行一个进程，即为```Init```进程。此进程在系统的整个生命周期一直运行，它管理加载你计算机所需要的其他进程。

既然大多数程序不会是```Init```进程，对进程启动来说就只剩下唯一一个实际的途径：```fork()```系统调用。当此函数被调用时，操作系统会复制进程并保持它们同时运行。原先的进程被称为”父进程“，复制出的新进程被称为”子进程“。```fork()```向子进程返回0，向父进程返回子进程的进程ID（PID）。本质上来说，开启新进程的唯一方法就是从已有的进程上复制一份。

这听起来似乎有个问题。技术上来说，当你想要启动一个新进程时，你并不是想要一个当前程序的复制品——你想要运行另一个程序。这就是```exec()```系统调用所要做的事情。（为方便理解，此句与原文有出入，出自[Wikipedia](https://zh.wikipedia.org/wiki/Fork_(%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8))——译者注）用其他程序覆盖自身：停止执行自己之前的程序并执行其他程序。这就意味着当你调用```exec()```后， 操作系统停止你的进程，加载新的程序，并启动它。一个进程永远不会从```exec()```调用返回（除非出现了错误）。

通过这两个系统调用，我们大概可以知道在Unix系统上大部分程序的运行方式了。首先，一个已存在的进程调用```fork```来创建一个自身的副本。然后，子进程调用```exec()```，用一个新的程序来替代自己。父进程可以继续运行下去，甚至可以通过系统调用```wait()```，来保持对子进程的跟踪。

呼！了解了这么多信息，但是根据这些背景知识，也使得接下来这个启动程序的代码是顺理成章的了：

```c
int lsh_launch(char **args)
{
  pid_t pid, wpid;
  int status;

  pid = fork();
  if (pid == 0) {
    // Child process
    if (execvp(args[0], args) == -1) {
      perror("lsh");
    }
    exit(EXIT_FAILURE);
  } else if (pid < 0) {
    // Error forking
    perror("lsh");
  } else {
    // Parent process
    do {
      wpid = waitpid(pid, &status, WUNTRACED);
    } while (!WIFEXITED(status) && !WIFSIGNALED(status));
  }

  return 1;
}
```

好了，此函数接收我们先前创建出的参数列表。然后，它分支出进程，并保存返回值。一旦```fork()```函数调用返回，我们就有了*两个*同时运行的进程了。第一个即为子进程如果满足情况（```pid``` == 0）的话。

在子进程中，我们希望运行用户提供的命令。所以，我们使用系统调用```exec```的众多变种之一，```execvp```。```exec```的不同变种提供了不同的功能，有些需要可变数量的字符串参数，有些则需要字符串列表。更有一些让你特别选定进程运行的环境。这一个（指execvp）特别的变种（第一个参数必须是程序名），需要一个程序名和一个字符串参数数组（也称为矢量，‘v')。’p‘意味着我们选择提供程序名，并让操作系统寻找环境路径（PATH）中的程序，而不是为了运行程序而提供完整的文件路径。

如果```execvp```命令返回 -1（或者它有返回状态的话），我们就知道有错误出现了。所以，我们使用```perror```来打印程序名和系统错误信息，这样用户就可以知道是哪儿出了错。然后，程序退出留待Shell继续运行。

第二种情况（pid < 0）检查```fork()```是否出现错误。如果出现了，我们将其打印出来并继续——除了告知用户此错误并由他们来决定是否需要退出以外，我们别无他法。

第三种情况意味着```fork()```成功执行。父进程将保持不变。我们知道子进程将要开始执行，所以父进程需要等待命令完成运行。我们使用```waitped()```来等待进程的状态改变。棘手的是，```waitpid()```有许多选项（正如```exec()```一样）。进程可能会以多种方式改变状态，并不是所有的方式都表示进程已经结束了。一个进程可能是退出了（普通情况下，或伴随一个错误码），也可能是被信号终止。所以，我们用```waitpid()```提供的宏来等待进程退出或是终止。然后，函数最终返回1，作为调用此函数的信号，我们应该再次提示输入。



## Shell内置函数



你可能已经注意到了，```lsh_loop()```函数循环中调用的是```lsh_execute()```函数，而我们却以```lsh_launch()```命名我们上述实现的函数。这是有意为之的！可以看到，Shell执行的大部分命令都是（外部）程序，但不是全部。其中有一些是Shell的内置函数。

这样做的理由很简单。（举例来说）如何你想要改变目录，你需要使用```chdir()```函数。然而，当前所在目录是进程的一个属性。所以，如果你写了一个名为```cd```的更改目录的程序，它只会改变它自己的目录，然后便终止了。它的父进程的目录并不会改变。然而，shell进程本身需要执行chdir（），以便更新自己的当前目录。这样，当它启动子进程时，子进程也可以继承该目录了。

类似的，如果有一个名为```exit```的程序，它理应不会使得调用它的Shell退出。同样的，对大多数的Shell来说，是通过运行类似```~/.bashrc```的配置脚本来对自身进行配置的。这些脚本使用改变Shell操作方式的命令。要做到这一点，只有在这些命令是Shell的内部实现的前提下才有可能。

如此一来，添加一些Shell的内置命令也就变得顺理成章了。我为自己的Shell选择的是```cd```，```exit```，和```help```。下面是它们的函数实现：

```c
/*
  Function Declarations for builtin shell commands:
 */
int lsh_cd(char **args);
int lsh_help(char **args);
int lsh_exit(char **args);

/*
  List of builtin commands, followed by their corresponding functions.
 */
char *builtin_str[] = {
  "cd",
  "help",
  "exit"
};

int (*builtin_func[]) (char **) = {
  &lsh_cd,
  &lsh_help,
  &lsh_exit
};

int lsh_num_builtins() {
  return sizeof(builtin_str) / sizeof(char *);
}

/*
  Builtin function implementations.
*/
int lsh_cd(char **args)
{
  if (args[1] == NULL) {
    fprintf(stderr, "lsh: expected argument to \"cd\"\n");
  } else {
    if (chdir(args[1]) != 0) {
      perror("lsh");
    }
  }
  return 1;
}

int lsh_help(char **args)
{
  int i;
  printf("Stephen Brennan's LSH\n");
  printf("Type program names and arguments, and hit enter.\n");
  printf("The following are built in:\n");

  for (i = 0; i < lsh_num_builtins(); i++) {
    printf("  %s\n", builtin_str[i]);
  }

  printf("Use the man command for information on other programs.\n");
  return 1;
}

int lsh_exit(char **args)
{
  return 0;
}
```

代码分为三个部分。第一部分包含函数的前置声明。前置声明使得你可以在定义它（而尚未实现时）以前使用它的名字。我这样做的原因是因为```lsh_help()```函数需要用到内置函数数组，而这个数组包含```lsh_help()```的函数名。解决这一依赖循环的最简洁办法就是使用前置声明了。

下一部分是一个包含内置命令名称的数组，及包含其相应函数的数组。这样实现的原因是，在未来，可以简单地通过修改此数组来添加内置命令，而不必通过编辑代码中某处一个大型的”switch“语句段。如果你对```builtin_func```的声明感到疑惑的话，没关系，我也如此！它是一个函数指针数组（接收字符串并返回一个```int```）。在C中，任何涉及到函数指针的声明都会引起困惑。我自己仍然在学习函数指针是如何声明的！

最后，我实现了每一个函数，```lsh_cd()```首先检查它接收的第二个参数是否存在，若否的话则打印错误信息。然后，它调用```chdir()```，检查是否有错误，返回。help函数打印帮助信息以及所有内置函数的名字。exit函数返回0，作为```loop```循环命令终止的信号。



## 将内置函数和进程整合起来



最后一片迷失的拼图就是实现```lsh_execute()```函数了，它将会启动一个内置函数，或者一个进程。如果你已经读到了这儿的话，你会发现这个函数的实现是十分简单的：

```c
int lsh_execute(char **args)
{
  int i;

  if (args[0] == NULL) {
    // An empty command was entered.
    return 1;
  }

  for (i = 0; i < lsh_num_builtins(); i++) {
    if (strcmp(args[0], builtin_str[i]) == 0) {
      return (*builtin_func[i])(args);
    }
  }

  return lsh_launch(args);
}
```

它所做的全部事情就是，检查输入命令是否是内置命令，如果是的话，运行它。否则，调用```lsh_launch()```启动程序进程。值得考虑的一点是，如果用户输入了空字符串，或者空格符，输入参数```args```就有可能为NULL。所以，我们需要在开头就检查这些情况。



## 组织代码



以上就是Shell中的全部代码了。如果你读到了这里，你应该已经完全明白Shell的工作原理了。为了尝试运行它（在一台Linux机器上），你应该将这些代码整合到一个文件中（```main.c```)，并且编译它。要注意其中只应包含一个```lsh_read_line()```函数的实现。你需要在文件顶部包含以下头文件。我添加了注释来帮助你了解每个函数的出处。

- ```
  #include <sys/wait.h>
  ```

  - `waitpid()` and associated macros

- ```
  #include <unistd.h>
  ```

  - `chdir()`
  - `fork()`
  - `exec()`
  - `pid_t`

- ```
  #include <stdlib.h>
  ```

  - `malloc()`
  - `realloc()`
  - `free()`
  - `exit()`
  - `execvp()`
  - `EXIT_SUCCESS`, `EXIT_FAILURE`

- ```
  #include <stdio.h>
  ```

  - `fprintf()`
  - `printf()`
  - `stderr`
  - `getchar()`
  - `perror()`

- ```
  #include <string.h>
  ```

  - `strcmp()`
  - ```strtok()```

一旦你已经编写好了代码和头文件，那么只需要运行```gcc -o main main.c```来编译它，然后使用```./main ```来运行就好了。

或者，你也可以在[Github](https://github.com/brenns10/lsh/tree/407938170e8b40d231781576e05282a41634848c)上找到这些代码。该链接直接指向编写本文时的当前修订版的代码——在未来的某一天我可能会考虑更新它并加上一些新功能。如果我这么做了，我会尽我所能地将这些细节和实现思路更新在文章中的。



## 总结



如果你已经读到了这里，并且好奇我究竟是怎么知道如何使用这些系统调用函数的，答案十分简单：```man```手册。在```man 3p```里，每个系统调用都有详尽的文档。如果你清楚你的需求，并且想知道如果去实现它，```man```手册将是你的绝佳助手。如果你不清楚C标准库和Unix系统给你提供了哪些接口，我向你推荐[POSIX  规范](http://pubs.opengroup.org/onlinepubs/9699919799/)，特别是**13章节**，”Headers“。你可以在那里找到每一个拥有详尽定义的头文件文档。

当然，这个Shell并不是功能完备的。明显的一些缺陷就包括：

- 仅允许使用空格分隔参数，不允许使用引用或是反斜杠转义。
- 没有管道或重定向。
- 仅有少量的内置函数。
- 不支持通配符。

上述功能的实现是十分有趣的事情，但那样已经远远超出我这篇文章所能容纳的内容范围了。如果我有机会去实现其中之一的话，我会写一篇后续文章的。但是我鼓励我的读者们去亲自实现这些功能。如果你成功了的话，请在评论区留言，我十分乐于见到这些代码。

最后，感谢阅读本篇教程（如果有人这么做了的话）。我享受写作本文的过程，希望你们能同样享受阅读它的过程。在评论区留下你们的想法吧！

**后记：**在文章的早期版本中，我在```lsh_split_line()```中遇到了一些烦人的错误，这些错误恰好导致了函数间相互抵消。感谢Reddit上的```/u/munmap```（还有其他一些评论用户）找到了这些错误！可在[diff](https://github.com/brenns10/lsh/commit/486ec6dcdd1e11c6dc82f482acda49ed18be11b5)中查看我犯了什么错误。

**后记2：**感谢Github用户ghswa为函数```malloc()```所做出的null检查贡献。他/她同样指出了```get_line()```函数的[manpage](http://pubs.opengroup.org/onlinepubs/9699919799/functions/getline.html)上表明其第一个参数应是可释放的，故而在我使用了```getline()```实现的```lsh_read_line()```函数中，```line```应被初始化为```NULL```。

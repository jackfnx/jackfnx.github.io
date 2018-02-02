title: 在C语言里用iconv转换字符编码（UTF8->GB18030）
author: sixue
tags:
  - C
  - iconv
  - ''
categories: []
date: 2018-02-03 01:39:00
---
iconv相关函数主要是三个

``` C
iconv_t iconv_open(const char* tocode, const char* fromcode);

size_t iconv(iconv_t cd, char **inbuf, size_t *inbytesleft, char **outbuf, size_t *outbytesleft);

int iconv_close(iconv_t cd);
```

这段代码的作用是从一个文本文件里读出字符串，转换一下编码，再写入另一个文件

``` C
#include <stdio.h>
#include <stdlib.h>
#include <memory.h>
#include <iconv.h>

const int LENGTH = 80;
const int BUFSZ = LENGTH * 2;

int print_n_str(const char *str, long len)
{
    char buf[BUFSZ];
    memset(buf, 0, BUFSZ);
    memcpy(buf, str, len);
    return printf("in:[%s](%ld)\n", buf, len);
}

int print_hex_str(const char *str, long len)
{
    printf("out:[");
    for (int i = 0; i < len; i++)
    {
        printf("%%%02x", (unsigned char)str[i]);
    }
    printf("](%ld)\n", len);
    return 0;
}

int main()
{
    iconv_t icv = iconv_open("GB18030", "UTF-8");
    FILE *in = fopen("in.txt", "r");
    FILE *out = fopen("out.txt", "w");
    
    char in_line[BUFSZ];
    char out_line[BUFSZ];
    size_t left = 0;
    while (!feof(in))
    {
        memset(in_line + left, 0, BUFSZ - left);
        fread(in_line + left, 1, LENGTH, in);
        left = strlen(in_line);
        
        char *inbuf = in_line;
        char *outbuf = out_line;
        size_t inleft = left;
        size_t outleft = BUFSZ;
        iconv(icv, &inbuf, &inleft, &outbuf, &outleft);
        
        size_t ilen = left - inleft;
        print_n_str(in_line, ilen);
        if (!inleft)
        {
            memmove(in_line, in_line + ilen, inleft);
        }
        left = inleft;
        
        size_t olen = BUFSZ - outleft;
        print_hex_str(out_line, olen);
        fwrite(out_line, 1, olen, out);
    }
    fclose(in);
    fclose(out);
    iconv_close(icv);
    return 0;
}
```

代码很简单，但实际上好几个坑：

1.函数1，两个参数是dest, src很容易无意中写错了，然后还发现不了

2.函数2，后面四个参数都是会变的，不要把原来的变量傻乎乎传进去到时候就找不回来了

3.函数2，有些时候我们的inbuf里不一定是完整的utf8字符串，可能有一些是被截断的“半个汉字”，此时iconv()会返回-1，并且会有errno，但是其实在应用层，这未必是错误，而是需要处理的情况。此时就需要inbytesleft参数，这个参数存的是剩下没处理的数据。

4.函数2，outbyetsleft指的是outbuf剩余的空闲空间，不要把它当成输出字符串的长度
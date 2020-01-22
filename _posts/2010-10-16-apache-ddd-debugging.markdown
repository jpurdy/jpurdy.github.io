---
layout: post
title: Debugging Apache Modules in Linux with GDB/DDD
date: 2010-10-16 07:07:07 +0300
description: Debugging Apache Modules in Linux with GDB/DDD
img: 2010-10-16-1.jpg
fig-caption:
tags: [Gzip, Linux]
---
This guide documents the steps required to debug an Apache 2.2.16 module. The module being debugged in this example is the deflate module (mod_deflate.c) as well as the zlib library which is uses to compress data with. Both the zlib library and deflate module, in this example, contain custom code with which we wish to step through.

1. Download and compile the Apache source distribution. Also make sure you don’t already have an Apache installation on your system. You can download the Apache source from [here](http://httpd.apache.org/download.cgi).
```
# EXTRA_CFLAGS=”-g” ./configure –prefix=/ap –with-included-apr –enable-mods-shared=all
# make
# make install
```
*Notes:*  
*EXTRA_CFLAGS=-g tells the compiler to include debug symbols.*  
*–prefix=/ap Places the install in /ap.*  
*–with-included-apr removes the possibility of version or compile-option mismatches with   APR and APR-util code (may not be necessary, but doesn’t hurt).*  
*–enable-mods-shared=all allows for the ability to alter a module, and then reload it. If this option is not used, the module code is compiled into the main Apache binary.*  
2. Update the Apache configuration file at /ap/config/httpd.conf.  
    Make sure the line `LoadModule deflate_module modules/mod_deflate.so` (or something similar) is present.  
    Add the line `AddOutputFilterByType DEFLATE text/html text/plain text/xml` (or something similar).  
3. Compile the zlib library (with the default compile flags in this case).  
```
# CFLAGS=”-g” ./configure –prefix=bin
In the Makefile remove the -03 option so that the code is not optimized.
# make test
# make install
```
*Notes:*  
>By default zlib builds a static library.
>EXTRA_CFLAGS=-g tells the compiler to include debug symbols.  
>–prefix=/ap Places the install in bin.  
4. Compile and install mod_deflate.  
```
# /ap/bin/apxs -I/mydir/zlib/bin/include/ -L/mydir/zlib/bin/lib/ -c mod_deflate.c -lahaz -g
# cp .libs/mod_deflate.so /ap/modules/mod_deflate.so
# /ap/bin/apachectl -k stop
# /ap/bin/apachectl -k start
```
*Notes:*  
>-g tells the compiler to include debug symbols.  
5. Start debugging
```
# ddd /ap/bin/httpd
        (gdb) r -X
        ctrl-c to return to the gdb prompt
        File->Open Source and select either the mod_deflate.c or aha363_zlib.c
        Set breakpoint visually or via the gdb command (i.e. (gdb) b aha363_zlib.c )
```

Notes: From The Apache Modules Book – Application Development with Apache pg 328 
>“.. we use the -X option to prevent Apache from detaching itself, forking children, and going into daemon mode… [Apache] is blocked while waiting for incoming connections. All modules are loaded, and the configuration is active. If we leave it there, the webserver is basically up and running and will service incoming request. We can interrupt it with Ctrl-c to return to the debugger.”  

That should be all that is required to get the Apache module code ready to debug.

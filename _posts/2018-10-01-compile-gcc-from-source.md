---
title: compile gcc from source
tags:
  - gcc
---

<!--more-->
# 源码编译gcc 7.3

1. ## 编译gmp

    https://gmplib.org/

    ```
    ./configure --prefix=/tool/gnu/gmp/6.1.2/
    make
    make install
    ```

2. ## 编译mpfr

    https://www.mpfr.org/

    ```
    ./configure --prefix=/tool/gnu/mpfr/4.0.1/ --with-gmp=/tool/gnu/gmp/6.1.2
    make
    make install   
    ```

3. ## 编译mpc

   https://ftp.gnu.org/gnu/mpc/

   ```
   ./configure --prefix=/tool/gnu/mpc/1.1.0/ --with-mpfr=/tool/gnu/mpfr/4.0.1 --with-gmp=/tool/gnu/gmp/6.1.2
   make
   make install
   ```

4. ## 编译isl

   http://isl.gforge.inria.fr/

   ```
   ./configure --prefix=/tool/gnu/isl/0.20/ --with-gmp=/tool/gnu/gmp/6.1.2
   make
   make install
   ```

5. ## 编译gcc 7.3

   https://ftp.gnu.org/gnu/gcc/

   ```
   ./configure --prefix=/tool/gnu/gcc/7.3.0/ --with-mpc=/tool/gnu/mpc/1.1.0 --with-mpfr=/tool/gnu/mpfr/4.0.1 --with-gmp=/tool/gnu/gmp/6.1.2 --with-isl=/tool/gnu/isl/0.20 --enable-languages=c,c++ --disable-multilib
   make
   make install
   ```

6. ##### 期间遇到的问题

   6.1 https://gcc.gnu.org/bugzilla/show_bug.cgi?id=85835

   6.2 https://gcc.gnu.org/bugzilla/show_bug.cgi?id=86724

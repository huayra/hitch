#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.59])
AC_INIT([hitch], [1.3.0~beta1], [support@varnish-software.com])
AC_CONFIG_SRCDIR([src/configuration.c])
AC_CONFIG_HEADERS([config.h])

AM_INIT_AUTOMAKE([foreign color-tests parallel-tests])

# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_MAKE_SET
AC_PROG_YACC
AC_PROG_LEX

AC_ARG_WITH([rst2man],
  AS_HELP_STRING([--with-rst2man=PATH], [Location of rst2man (auto)]),
  [RST2MAN="$withval"],
  AC_CHECK_PROGS(RST2MAN, [rst2man rst2man.py], [no]))
if test "x$RST2MAN" = "xno"; then
  AC_MSG_ERROR(
    [rst2man is needed to build Hitch, please install python-docutils. To proceed without building man pages, specify --with-rst2man=/bin/true.])
fi

AM_MAINTAINER_MODE([disable])

# Checks for libraries.
PKG_CHECK_EXISTS([libev], [
  PKG_CHECK_MODULES([EV], [libev])], [
  AC_SEARCH_LIBS([ev_default_loop], [ev], [], AC_MSG_ERROR([Cannot find libev.]))
])

PKG_CHECK_MODULES([SSL], [libssl])
PKG_CHECK_MODULES([CRYPTO], [libcrypto])

AC_CHECK_MEMBERS([struct stat.st_mtim, struct stat.st_mtimespec])

AC_ARG_ENABLE(sessioncache,
    AC_HELP_STRING([--enable-sessioncache],
		   [Enable TLS session cache. (default is off)]),
    [use_shctx="$withval"],
    [use_shctx=no])
AM_CONDITIONAL(USE_SHCTX, test xno != x"$use_shctx")

# Checks for header files.
AC_CHECK_HEADERS([stdlib.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_TYPE_UID_T
AC_C_INLINE
AC_TYPE_INT32_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UINT32_T

# Checks for library functions.
AC_FUNC_FORK
AC_FUNC_MMAP
AC_CHECK_FUNCS([accept4])

SH_TESTS="$(cd src && echo tests/test*)"
AC_SUBST(SH_TESTS)

AC_CONFIG_FILES([Makefile])
AC_CONFIG_FILES([src/Makefile])

AC_OUTPUT

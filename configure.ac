# configure.ac
#
# Copyright (C) 2019-2024 wolfSSL Inc.
#

AC_COPYRIGHT([Copyright (C) 2024 wolfSSL Inc.])
AC_PREREQ([2.69])
AC_INIT([wolfprov], [1.0.2])
AC_CONFIG_AUX_DIR([build-aux])

AC_CONFIG_HEADERS([include/config.h])
AC_CONFIG_MACRO_DIR([m4])

dnl m4_include([m4/ax_check_openssl.m4])
dnl m4_include([m4/ax_check_wolfssl.m4])
m4_include([m4/hexversion.m4])

AM_INIT_AUTOMAKE([1.11  -Wall -Werror -Wno-portability foreign tar-ustar subdir-objects no-define color-tests])
LT_INIT([disable-static pic-only])

# capture user C_EXTRA_FLAGS from ./configure line, CFLAGS may hold -g -O2 even
# if user doesn't override, no way to tell
USER_C_EXTRA_FLAGS="$C_EXTRA_FLAGS"
USER_CFLAGS="$CFLAGS"

AC_PROG_CC
AC_LANG(C)

# OpenSSL
AC_CHECK_HEADERS([openssl/core.h])

AX_CHECK_OPENSSL(
    [have_openssl=yes],
    AC_MSG_FAILURE([could not locate OpenSSL])
)

if test "x$have_openssl" = "xyes"; then
    LIBS="$LIBS $OPENSSL_LIBS"
    CFLAGS="$CFLAGS $OPENSSL_INCLUDES"
    LDFLAGS="$LDFLAGS $OPENSSL_LDFLAGS"
fi

# wolfSSL
AX_CHECK_WOLFSSL(
    [have_wolfssl=yes],
    AC_MSG_FAILURE([could not locate wolfSSL])
)

if test "x$have_wolfssl" = "xyes"; then
    LIBS="$LIBS $WOLFSSL_LIBS"
    CFLAGS="$CFLAGS $WOLFSSL_INCLUDES"
    LDFLAGS="$LDFLAGS $WOLFSSL_LDFLAGS"
fi

# Check for required headers
AC_CHECK_HEADERS([stdlib.h])
AC_CHECK_HEADERS([unistd.h])
AC_CHECK_HEADERS([ctype.h])

# Check for getpid function and add HAVE_GETPID to AM_CFLAGS for inclusion in options.h
AC_CHECK_FUNC([getpid])
if test "$ac_cv_func_getpid" = "yes"
then
    AM_CFLAGS="$AM_CFLAGS -DHAVE_GETPID=1"
fi

# DEBUG
DEBUG_CFLAGS="-g -O0 -DWOLFPROV_DEBUG"

# DEBUG
AX_DEBUG
AS_IF([test "$ax_enable_debug" = "yes"],
      [AM_CFLAGS="$DEBUG_CFLAGS $AM_CFLAGS"],
      [])

# COVERAGE
COVERAGE_CFLAGS="--coverage"
AX_COVERAGE
AS_IF([test "$ax_enable_coverage" = "yes"],
      [AM_CFLAGS="$COVERAGE_CFLAGS $AM_CFLAGS"])

# user_settings.h
AC_ARG_ENABLE([usersettings],
    [AS_HELP_STRING([--enable-usersettings],[Use your own user_settings.h and do not add Makefile CFLAGS (default: disabled)])],
    [ ENABLED_USERSETTINGS=$enableval ],
    [ ENABLED_USERSETTINGS=no ]
    )

# Dynamic provider support
AC_ARG_ENABLE([dynamic],
    [AS_HELP_STRING([--enable-dynamic],[Enable loading wolProv as a dynamic provider (default: enabled).])],
    [ ENABLED_DYNAMIC_PROVIDER=$enableval ],
    [ ENABLED_DYNAMIC_PROVIDER=yes ]
    )

if test "$ENABLED_DYNAMIC_PROVIDER" != "yes"
then
    AM_CFLAGS="$AM_CFLAGS -DWP_NO_DYNAMIC_PROVIDER"
fi

# Single threaded
AC_ARG_ENABLE([singlethreaded],
    [AS_HELP_STRING([--enable-singlethreaded],[Enable wolfEngine single threaded (default: disabled).])],
    [ ENABLED_SINGLETHREADED=$enableval ],
    [ ENABLED_SINGLETHREADED=no ]
    )

if test "$ENABLED_SINGLETHREADED" = "yes"
then
    AM_CFLAGS="$AM_CFLAGS -DWP_SINGLE_THREADED"
fi

AS_IF([ test "x$ENABLED_SINGLETHREADED" = "xno" ],[
       AX_PTHREAD([
                   AC_DEFINE([HAVE_PTHREAD], [1], [Define if you have POSIX threads libraries and header files.])
                   # If AX_PTHREAD is adding -Qunused-arguments, need to prepend with -Xcompiler libtool will use it. Newer
                   # versions of clang don't need the -Q flag when using pthreads.
                   AS_CASE([$PTHREAD_CFLAGS],[-Qunused-arguments*],[PTHREAD_CFLAGS="-Xcompiler $PTHREAD_CFLAGS"])
                   AM_CFLAGS="-D_POSIX_THREADS $AM_CFLAGS $PTHREAD_CFLAGS"
                   LIBS="$LIBS $PTHREAD_LIBS"
                   ],[
                      ENABLED_SINGLETHREADED=yes
                      ])
      ])





AX_HARDEN_CC_COMPILER_FLAGS

OPTION_FLAGS="$CFLAGS $CPPFLAGS $AM_CFLAGS"

LIBS="$LIBS -lwolfssl -ldl -lm"

if test "$GCC" = "yes"
then
    AM_CFLAGS="$AM_CFLAGS -Wall -Wno-unused -Wno-error=deprecated-declarations"
fi

CFLAGS="$CFLAGS $USER_C_EXTRA_FLAGS"

AC_SUBST([AM_CPPFLAGS])
AC_SUBST([AM_CFLAGS])
AC_SUBST([AM_LDFLAGS])
CREATE_HEX_VERSION

AC_CONFIG_FILES([Makefile
        include/wolfprovider/version.h
        ])
AC_OUTPUT

# force make clean
echo "---"
echo "Running make clean..."
make clean >/dev/null 2>&1


# output config summary
echo "---"
echo "Configuration summary for $PACKAGE_NAME version $VERSION"
echo ""
echo "   * Installation prefix:        $prefix"
echo "   * System type:                $host_vendor-$host_os"
echo "   * Host CPU:                   $host_cpu"
echo "   * C Compiler:                 $CC"
echo "   * C Flags:                    $CFLAGS"
echo "   * Debug enabled:              $ax_enable_debug"
echo
echo "   Features "
echo "   * User settings:              $ENABLED_USERSETTINGS"
echo "   * Dynamic provider:           $ENABLED_DYNAMIC_PROVIDER"
echo ""
echo "---"


dnl
dnl $Id: configure.in,v 1.3 2004/10/07 01:23:25 jnathan Exp $
dnl
dnl configure.in
dnl
dnl Copyright (C) 2002, 2003 Jeff Nathan <jeff@snort.org>
dnl
dnl Suggested reading:  "GNU Autoconf, Automake and Libtool" New Riders Press

AC_PREREQ(2.61)
AC_INIT(nemesis, 1.5-beta1, https://github.com/troglobit/nemesis/issues,, http://troglobit.com/projects/nemesis/)
AM_INIT_AUTOMAKE([1.11 foreign no-dist-gzip dist-xz])
AM_SILENT_RULES([yes])

AC_CONFIG_SRCDIR([src/nemesis.c])
AC_CONFIG_HEADER([config.h])
AC_CONFIG_FILES([Makefile src/Makefile man/Makefile])
AC_CONFIG_MACRO_DIR([m4])

AH_TOP(
#undef PACKAGE
#undef VERSION
#undef FREEBSD
#undef LINUX
#undef MACOS
#undef OPENBSD
#undef NETBSD
#undef SOLARIS
)

dnl Determine host type before performing further tests.
AC_CANONICAL_HOST

dnl Set the default prefix.
if test "$prefix" = "NONE"; then
    prefix="/usr/local"
fi

AM_PROG_CC_STDC
AC_PROG_CC
AC_PROG_GCC_TRADITIONAL
if test -n "$GCC"; then
    CFLAGS="${CFLAGS} -Wall"
fi

dnl Required versions
LIBNET_REQ=1.1.6

dnl Configure options
AC_ARG_ENABLE(debug,
    [  --enable-debug          enable debugging options (bugreports and developers)],
        [ if test -n "$GCC"; then
              CFLAGS="-g -O0 -Wall"
          fi
          CPPFLAGS="${CPPFLAGS} -DDEBUG" ],)

AC_ARG_ENABLE(profile,
    [  --enable-profile        enable profiling options (developers only)],
        [ if test -n "$GCC"; then
              CFLAGS="${CFLAGS} -pg -static"
          fi ],)

case "$host" in
  *-linux*)
    AC_DEFINE(LINUX)	
    ;;

  *-freebsd*)
    AC_DEFINE(FREEBSD)
    ;;

  *-openbsd*)
    AC_DEFINE(OPENBSD)
    ;;

  *-netbsd*)
    AC_DEFINE(NETBSD)
    ;;

  *-solaris*)
    AC_DEFINE(SOLARIS)
    ;;

  *-apple*)
    AC_DEFINE(MACOS)
    LIBS="${LIBS} -L/sw/lib"
    CPPFLAGS="${CPPFLAGS} -I/sw/include"
    ;;

  *)
    AC_MSG_WARN(Your OS is not officially supported yet)
    AC_MSG_WARN(Nemesis may not work properly)
    ;;
esac

AC_ARG_WITH(libnet_includes,
    [  --with-libnet-includes=DIR    libnet include directory],
    [with_libnet_includes="$withval"],
    [with_libnet_includes=no])

AC_ARG_WITH(libnet_libraries,
    [  --with-libnet-libraries=DIR	libnet library directory],
    [with_libnet_libraries="$withval"],
    [with_libnet_libraries=no])

if test "$with_libnet_includes" != "no"; then
   CPPFLAGS="${CPPFLAGS} -I${with_libnet_includes}"
fi   

if test "$with_libnet_libraries" != "no"; then
   LIBS="${LIBS} -L${with_libnet_libraries}"
fi

CPPFLAGS="${CPPFLAGS} `libnet-config --defines`"
CFLAGS="${CFLAGS} -funroll-loops -pipe"

if test `libnet-config --cflags | wc -c` = "1"; then
    CPPFLAGS="${CPPFLAGS} -I/usr/local/include -I/sw/include"
    LIBNET_CONFIG_BROKEN_CFLAGS=yes
else
    CPPFLAGS="${CPPFLAGS} `libnet-config --cflags`"
fi

if test `libnet-config --libs | wc -c` = "1"; then
    AC_MSG_WARN(libnet-config --libs is broken on your system.  If you)
    AC_MSG_WARN(are using a precompiled package please notify the maintainer.)
    LIBS="${LIBS} -L/usr/local/lib -L/sw/lib"
else
    LIBS="${LIBS} `libnet-config --libs | sed s/\-lnet//`"
fi

AC_DEFUN([HEADER_FAIL_MESSAGE],[
   echo
   echo "************************************************************************"
   echo "  ERROR: unable to find" $1
   echo "  checked in the following places"
   for i in `echo $2`; do
       echo "        $i"
   done  
   echo "************************************************************************"
   echo
   exit 1
])

AC_DEFUN([FUNCTION_FAIL_MESSAGE],[
   echo
   echo "************************************************************************"
   echo "  ERROR: Nemesis cannot compile without"
   echo "         the function $1"
   echo "************************************************************************"
   echo
   exit 1
])

AC_C_CONST

dnl Program tests
AC_PROG_AWK
AC_PROG_INSTALL
AC_PROG_LN_S

dnl Library tests
AC_CHECK_LIB(m, fabs)
AC_CHECK_LIB(nsl, inet_ntoa)
AC_CHECK_LIB(socket, socket)
AC_CHECK_LIB(resolv, hstrerror)

LNET=""
AC_CHECK_LIB(net, libnet_build_ipv4,, LNET="no")

if test "$LNET" = "no"; then
    echo
    echo "   ERROR: Libnet library not found, get it from"
    echo "   https://sourceforge.net/projects/libnet-dev/"
    echo "   or use the --with-libnet-* options, if you have it installed"
    echo "   in unusual place"
    exit
fi

dnl Checks for header files.
AC_HEADER_STDC
AC_HEADER_TIME
dnl AC_CHECK_HEADERS(unistd.h errno.h strings.h netdb.h limits.h machine/endian.h sys/time.h sys/param.h arpa/inet.h netinet/in.h netinet/ip_var.h)
AC_CHECK_HEADERS(unistd.h errno.h strings.h netdb.h limits.h machine/endian.h sys/time.h sys/param.h arpa/inet.h netinet/in.h)

LNET=""
AC_CHECK_HEADERS(libnet.h,, LNET="no")

if test "$LNET" = "no"; then
   echo
   echo "   ERROR: Libnet header not found, get it from"
   echo "   https://sourceforge.net/projects/libnet-dev/"
   echo "   or use the --with-libnet-* options, if you have it installed"
   echo "   in unusual place"
   exit
fi

AC_MSG_CHECKING(for libnet version >= ${LIBNET_REQ})
if test "${LIBNET_CONFIG_BROKEN_CFLAGS}" = "yes"; then
    if test -n "$with_libnet_includes" -a "$with_libnet_includes" != "no"; then
        libnet_dir="${with_libnet_includes}"
    else
        libnet_dir="/usr/include /usr/local/include /sw/include /opt/include"
    fi
else
    libnet_dir=`libnet-config --cflags | cut -dI -f2`
fi

LIBNET_INC_DIR=""
for i in $libnet_dir; do
    if test -r $i/libnet.h; then
        LIBNET_INC_DIR=$i
    fi
done

if test "$LIBNET_INC_DIR" != ""; then
    VER=`grep LIBNET_VERSION $LIBNET_INC_DIR/libnet.h | sed 's/.*"\(.*\)"/\1/'`
    AX_COMPARE_VERSION([$VER], [lt], [${LIBNET_REQ}], [
        AC_MSG_RESULT(no)
        echo
        echo "   ERROR: Nemesis requires libnet >= ${LIBNET_REQ}, get it from:"
        echo "   https://sourceforge.net/projects/libnet-dev/"
        HEADER_FAIL_MESSAGE("libnet ${LIBNET_REQ} (libnet.h)", $LIBNET_INC_DIR)],[
        AC_MSG_RESULT(yes)])
else
    AC_MSG_RESULT(no)
    HEADER_FAIL_MESSAGE("libnet ${LIBNET_REQ} (libnet.h)", $libnet_dir)
fi

AC_TYPE_SIZE_T
AC_STRUCT_TM

dnl Checks for library functions.
AC_CHECK_FUNCS(gethostbyname,,FUNCTION_FAIL_MESSAGE("gethostbyname"))
AC_CHECK_FUNCS(gettimeofday,,FUNCTION_FAIL_MESSAGE("gettimeofday"))
AC_CHECK_FUNCS(inet_ntoa,,FUNCTION_FAIL_MESSAGE("inet_ntoa"))
AC_CHECK_FUNCS(isascii,,FUNCTION_FAIL_MESSAGE("isascii"))
AC_FUNC_MEMCMP
AC_CHECK_FUNCS(memset,,FUNCTION_FAIL_MESSAGE("memset"))
AC_CHECK_FUNCS(strchr,,FUNCTION_FAIL_MESSAGE("strchr"))
AC_CHECK_FUNCS(strdup,,FUNCTION_FAIL_MESSAGE("strdup"))
AC_CHECK_FUNCS(strerror,,FUNCTION_FAIL_MESSAGE("strerror"))
AC_CHECK_FUNCS(strrchr,,FUNCTION_FAIL_MESSAGE("strrchr"))
AC_CHECK_FUNCS(strtoul,,FUNCTION_FAIL_MESSAGE("strtoul"))

dnl Check for usually missing API's, which we can replace
AC_REPLACE_FUNCS(getopt inet_aton memcmp strlcat strlcpy strsep)
AC_CONFIG_LIBOBJ_DIR([lib])

CFLAGS=`echo $CFLAGS | sed -e 's/-I\/usr\/include //g'`
CPPFLAGS=`echo $CPPFLAGS | sed -e 's/-I\/usr\/include //g'`

AC_OUTPUT

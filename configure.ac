AC_PREREQ(2.57)
AC_INIT(Judy, 1.0.5, dougbaskins@yahoo.com)

AM_MAINTAINER_MODE

# Turn on automake, and pass it the PACKAGE_NAME and PACKAGE_VERSION, too.
AM_INIT_AUTOMAKE($PACKAGE_NAME, $PACKAGE_VERSION)

# Tell autoconf we want to keep our preprocessor defines in a header named
# config.h.  This keeps automake from passing a zillion -D directives to
# the C compiler.
AM_CONFIG_HEADER([config.h])

#==========================================================================
# WARNING - WARNING - Shared Library Versioning - WARNING - WARNING
#==========================================================================
# This is the most dangerous part of this file--making a mistake here can
# cause massively painful chaos for libJudy developers, and potentially
# even end users.  So PLEASE pay attention, and read up on the theory of
# shared library versioning.  Tens of thousands of Linux users (and several
# QA departments) may thank you someday.
#
# There are two major concerns:
#
#   1) When changing the libJudy ABI (application binary interface),
#      VERSION_INFO *must* be updated according to libtool's rules.  Failure
#      to do this will make applications using libJudy dump core, typically
#      under obscure conditions on user systems.  I won't attempt to
#      explain these rules here; please see 'info libtool' for details.
#
#   2) When changing the libJudy ABI, it is also desirable to make libJudy
#      "parallel installable".  This means that it should be possible to
#      install development headers and libraries for more than one version
#      of libJudy at once.  Failure to do this may cause problems for
#      Linux distributions which include libJudy.  (For example, it's
#      impossible to switch between libpng2-dev and libpng3-dev on a
#      Debian system without uninstalling and reinstalling both the Gnome
#      and KDE SDKs.)  For more information, do a Google search for
#      "parallel installable".
#
# Right now, this package only provides the mechanisms to handle concern
# (1).  Concern (2) is slightly more complicated, and will require some
# careful thinking.  Fortunately, concern (2) doesn't become important
# until other SDKs rely on the libJudy SDK.
#
# Of course, it's safe to avoid changing the libJudy ABI. :-)
#
# The version scheme used by Libtool tracks interfaces, where an interface is
# the set of exported entry points into the library. All Libtool libraries
# start with -version-info set to 0:0:0 - this will be the default version
# number if you don't explicitly set it on the Libtool link command line. The
# meaning of these numbers (from left to right) is as follows:
#
#  current:
#     The number of the current interface exported by the library. A current
#     value of 0, means that you are calling the interface exported by this
#     library interface 0.
#
#  revision:
#     The implementation number of the most recent interface exported by this
#     library. In this case, a revision value of 0 means that this is the
#     first implementation of the interface.
#
#     If the next release of this library exports the same interface, but has
#     different implementation (perhaps some bugs have been fixed), the
#     revision number will be higher, but current number will be the same. In
#     that case, when given a choice, the library with the highest revision
#     will always be used by the runtime loader.
#
#   age:
#     The number of previous additional interfaces supported by this library.
#     If age were 2, then this library can be linked into executables which
#     were built with a release of this library that exported the current
#     interface number, current, or any of the previous two interfaces.
#
# By definition age must be less than or equal to current. At the outset, only
# the first ever interface is implemented, so age can only be 0.
#

VERSION_INFO="-version-info 1:3:0"
AC_SUBST(VERSION_INFO)

#==========================================================================
# Flavors
#==========================================================================
# Judy can be compiled in one of three flavors: "product" (the default),
# "debug", or "cov".  We allow the user to select flavors using
# --enable-debug and --enable-ccover arguments to automake, which is
# the typical way of doing things.
#
# Note how we perform string comparison:
#
#   if test "x$enable_debug" = xyes; then
#
# We do several odd things here:
#
#   1) We use 'test' instead of '[ ]' for shell portability.
#   2) We prefix strings with 'x' when comparing them, to protect against
#      empty strings.
#   3) We ALWAYS quote user-supplied shell variables, to protect against
#      embedded spaces.
#
# The results of this test aren't used anywhere yet.

# Keep the user entertained.
AC_MSG_CHECKING(which flavor to build)

# Process our --enable-debug argument.
AC_ARG_ENABLE(debug,
              AC_HELP_STRING([--enable-debug],
                             [enable debugging features]),
              , enable_debug=no)
if test "x$enable_debug" != xyes -a "x$enable_debug" != xno; then
   AC_MSG_ERROR(You may not pass an argument to --enable-debug)
fi

# Process our --enable-ccover argument.
AC_ARG_ENABLE(ccover,
              AC_HELP_STRING([--enable-ccover],
                             [enable use of ccover code coverage tools]),
              , enable_ccover=no)
if test "x$enable_ccover" != xyes -a "x$enable_ccover" != xno; then
   AC_MSG_ERROR(You may not pass an argument to --enable-ccover)
fi

# Determine our flavor.
if test "x$enable_debug" = xyes -a "x$enable_ccover" = xyes; then
   AC_MSG_ERROR(You may not use --enable-debug and --enable-ccover together)
elif test "x$enable_debug" = xyes; then
   FLAVOR=debug
elif test "x$enable_ccover" = xyes; then
   FLAVOR=cov
else
   FLAVOR=product
fi

# Define FLAVOR in our makefiles.
AC_SUBST(FLAVOR)

# Tell the user what flavor we've decided to build.
AC_MSG_RESULT($FLAVOR)


#==========================================================================
# Checks for Programs
#==========================================================================
AC_PROG_CC
AC_PROG_CPP
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET

#==========================================================================
# Checks for Header Files
#==========================================================================
AC_HEADER_STDC
AC_CHECK_HEADERS([fcntl.h inttypes.h limits.h malloc.h stddef.h stdint.h stdlib.h string.h strings.h sys/param.h sys/time.h unistd.h])

#==========================================================================
# Checks for Typedefs, Structures, and Compiler Characteristics
#==========================================================================

# Standard, boring stuff.
AC_HEADER_STDBOOL
AC_C_CONST
AC_C_INLINE
AC_TYPE_SIZE_T
AC_HEADER_TIME
AC_STRUCT_TM
AC_C_VOLATILE
AC_CHECK_TYPES([ptrdiff_t])

# If we're compiling for a little-endian system, define JU_LITTLE_ENDIAN.
# If we can't tell what kind of system we're compling for, alert the
# user as described in 'info autoconf'.
AC_C_BIGENDIAN(, AC_DEFINE(JU_LITTLE_ENDIAN, 1,
			   [Define to 1 on little-endian systems.]))

#  Figure out if we are 32-bit or 64-bit (LP64)
AC_CHECK_SIZEOF(void *)
if test "$ac_cv_sizeof_void_p" = 8; then
    AC_MSG_RESULT(Detected 64-bit Build Environment)
    CFLAGS="-DJU_64BIT $CFLAGS"
else
    AC_MSG_RESULT(Detected 32-bit Build Environment)
      CFLAGS="-UJU_64BIT $CFLAGS"
fi

AC_ARG_ENABLE(32-bit, [  --enable-32-bit          Generate code for a 32-bit environment],
	      b32="$enableval", b32="no")
if test x"$b32" != "xno"; then
    AC_MSG_RESULT(Configured to Build 32-bit)
    if test x"$GCC" = xyes; then
      CFLAGS="-UJU_64BIT -m32 $CFLAGS"
    else
      CFLAGS="-UJU_64BIT $CFLAGS"
    fi
fi

AC_ARG_ENABLE(64-bit, [  --enable-64-bit          Generate code for a 64-bit environment],
              b64="$enableval", b64="no")
if test x"$b64" != "xno"; then
    AC_MSG_RESULT(Configured to Building 64-bit)
    if test x"$GCC" = xyes; then
      CFLAGS="-DJU_64BIT -m64 $CFLAGS"
    else
      CFLAGS="-DJU_64BIT $CFLAGS"
    fi
fi



#  dnl Determine whether or not we're compiling for a 64-bit system by looking
#  dnl at the size of a 'long'.  This will define SIZEOF_LONG in config.h.  We
#  dnl append some text to the bottom of config.h to set JU_64BIT appropriately.
#  dnl we try to do the correct thing if the user doesn't chose for us. 
#  AC_CHECK_SIZEOF(long)
#  AH_BOTTOM([/* Define JU_64BIT to 1 if we're on a 64-bit system. */
#  if SIZEOF_LONG == 8
#    define JU_64BIT 1
#  endif])
#fi

#==========================================================================
# Checks for Libraries
#==========================================================================

AC_FUNC_ERROR_AT_LINE
AC_FUNC_MALLOC
AC_FUNC_MEMCMP
AC_FUNC_MMAP
AC_FUNC_STAT
AC_FUNC_VPRINTF
AC_CHECK_FUNCS([getpagesize gettimeofday memset munmap pow strchr strcspn strerror strstr strtoul uname])

# These must be called before AM_PROG_LIBTOOL, because it may want
# to call AC_CHECK_PROG.
AC_CHECK_TOOL(AR, ar)
AC_CHECK_TOOL(LD, ld)
AC_CHECK_TOOL(RANLIB, ranlib, :)

# Checks for libtool - this must be done after we set cflags (abi issues)
#
AM_PROG_LIBTOOL

WARN_CFLAGS=""
build_warnings="-W -Wall -Wstrict-prototypes -Wmissing-prototypes"
AC_ARG_ENABLE([build-warnings],
	      [  --enable-build-warnings    Enable build-time compiler warnings for gcc])
if test x"$build_warnings" = xyes; then
    if test x"$GCC" = xyes; then
	WARN_CFLAGS="${build_warnings}"
    fi
fi

AC_SUBST(WARN_CFLAGS)

AC_CONFIG_FILES([Makefile
                 src/Judy1/Makefile
                 src/JudyCommon/Makefile
                 src/JudyHS/Makefile
                 src/JudyL/Makefile
                 src/JudySL/Makefile
                 src/Makefile
                 src/obj/Makefile
		 tool/Makefile
		 doc/Makefile
		 test/Makefile])


AC_OUTPUT

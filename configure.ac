AC_PREREQ(2.57)
AC_INIT([Judy],[1.0.5])
AC_CONFIG_SRCDIR([src/Judy.h])
AM_INIT_AUTOMAKE
AC_CONFIG_MACRO_DIRS([m4])
AC_CONFIG_HEADERS([config.h])

AM_MAINTAINER_MODE

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
FLAVOR=product

# Process our --enable-debug argument.
AC_ARG_ENABLE([debug],
	      AS_HELP_STRING([--enable-debug],
			     [enable debugging features @<:@default=no@:>@]))

# Process our --enable-ccover argument.
AC_ARG_ENABLE([ccover],
	      AS_HELP_STRING([--enable-ccover],
			     [enable use of ccover code coverage tools @<:@default=no@:>@]))

# Determine our flavor.
if test x$enable_debug = xyes -a x$enable_ccover = xyes; then
    AC_MSG_ERROR([You may not use --enable-debug and --enable-ccover together])
elif test x$enable_debug = xyes; then
    FLAVOR=debug
elif test x$enable_ccover = xyes; then
    FLAVOR=cov
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
AC_C_BIGENDIAN(,AC_DEFINE(JU_LITTLE_ENDIAN,1,[Define to 1 on little-endian systems.]))

AC_ARG_ENABLE([32bit],
	      AS_HELP_STRING([--enable-32bit],
			     [Generate code for a 32bit environment @<:@default=auto@:>@]))

AC_ARG_ENABLE([64bit],
	      AS_HELP_STRING([--enable-64bit],
			     [Generate code for a 32bit environment @<:@default=auto@:>@]))

FORCE_ARCH=auto
if test x$enable_debug = xyes -a x$enable_ccover = xyes; then
    AC_MSG_WARN([Both --enable-debug and --enable-ccover, both ignored.])
elif test x$enable_32bit = xyes; then
    AC_MSG_RESULT(Force generate code for a 32bit environment.)
    FORCE_ARCH=m32
    CFLAGS="-UJU_64BIT $CFLAGS"
elif test x$enable_64bit = xyes; then
    AC_MSG_RESULT(Force generate code for a 64bit environment.)
    FORCE_ARCH=m64
    CFLAGS="-DJU_64BIT $CFLAGS"
fi

if test x$FORCE_ARCH = xauto; then
    #  Figure out if we are 32-bit or 64-bit (LP64)
    AC_CHECK_SIZEOF(void *)
    case x$ac_cv_sizeof_void_p in
	x4)
	    AC_MSG_RESULT([Detected 32bit build environment])
	    CFLAGS="-UJU_64BIT $CFLAGS"
	    ;;
	x8)
	    AC_MSG_RESULT([Detected 64bit build environment])
	    CFLAGS="-DJU_64BIT $CFLAGS"
	    ;;
	*)
	    AC_MSG_ERROR([Unrecorgnized build environment: x$ac_cv_sizeof_void_p])
    esac
elif test x$GCC = xyes; then
    CFLAGS="-${FORCE_ARCH} $CFLAGS"
fi

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
AC_CHECK_TOOL([AR],[ar])
AC_CHECK_TOOL([LD],[ld])
AC_CHECK_TOOL([RANLIB],[ranlib],[:])

# Checks for libtool - this must be done after we set cflags (abi issues)
#
AM_PROG_LIBTOOL

WARN_CFLAGS=""
build_warnings="-W -Wall -Wstrict-prototypes -Wmissing-prototypes"

AC_ARG_ENABLE([warnings],
	      AS_HELP_STRING([--enable-warnings],
			     [Enable build-time compiler warnings for gcc @<:@default=no@:>@]))

if test x$enable_warnings = xyes -a x$GCC = xyes; then
    WARN_CFLAGS="${build_warnings}"
fi

AC_SUBST(WARN_CFLAGS)


AC_ARG_ENABLE([manpage],
	      AS_HELP_STRING([--enable-manpage],
			     [Enable build manpage @<:@default=no@:>@]))
AM_CONDITIONAL([BUILD_MANPAGE],[test x$enable_manpage = xyes])


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

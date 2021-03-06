#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

m4_define(pkg_major_version, 1)
m4_define(pkg_minor_version, 33)
m4_define(pkg_micro_version, 2)
m4_define(pkg_version, pkg_major_version.pkg_minor_version.pkg_micro_version)

AC_PREREQ(2.61)
AC_INIT([gjs], pkg_version, [http://bugzilla.gnome.org/enter_bug.cgi?product=gjs])
AM_INIT_AUTOMAKE([dist-bzip2 no-dist-gzip])
AC_CONFIG_SRCDIR([gjs/console.c])
AC_CONFIG_HEADER([config.h])

GETTEXT_PACKAGE=gjs
AC_SUBST([GETTEXT_PACKAGE])
AC_DEFINE_UNQUOTED([GETTEXT_PACKAGE], ["$GETTEXT_PACKAGE"], [The name of the gettext domain])

AM_MAINTAINER_MODE([enable])
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

# our first pkg-config invocation is conditional, ensure macros still work
PKG_PROG_PKG_CONFIG

AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_CXX
AC_ISC_POSIX
AC_HEADER_STDC

# no stupid static libraries
AM_DISABLE_STATIC
# avoid libtool for LTCOMPILE, use it only to link
AC_PROG_LIBTOOL
dnl DOLT

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST

# Add extra warning flags
changequote(,)dnl
ensureflag() {
  flag="$1"; shift
  result="$@"

  case " ${result} " in
  *[\ \	]${flag}[\ \	]*) ;;
  *) result="${flag} ${result}" ;;
  esac

  echo ${result}
}
changequote([,])dnl

if test "$GCC" = "yes"; then
    for flag in -Wall -Wchar-subscripts -Wmissing-declarations \
        -Wpointer-arith -Wcast-align -Wsign-compare;
    do
        CFLAGS="`ensureflag $flag $CFLAGS`"
        CXXFLAGS="`ensureflag $flag $CXXFLAGS`"
    done
    CFLAGS="`ensureflag -Wmissing-prototypes $CFLAGS`"
    CFLAGS="`ensureflag -Wnested-externs $CFLAGS`"
fi

# coverage
AC_ARG_ENABLE([coverage],
              [AS_HELP_STRING([--enable-coverage],
                              [enable code coverage])],
              , [enable_coverage=no])
if test x$enable_coverage = xyes; then
    if test x$GCC != xyes; then
        AC_MSG_ERROR([GCC is required for --enable-coverage])
    fi

    AC_PATH_TOOL([LCOV], [lcov])
    AC_PATH_TOOL([GENHTML], [genhtml])

    if test x$LCOV = x || test x$GENHTML = x; then
        AC_MSG_ERROR([lcov and genhtml are required for --enable-coverage])
    fi

    CFLAGS="$CFLAGS -g -O0 -fprofile-arcs -ftest-coverage"
    CXXFLAGS="$CXXFLAGS -g -O0 -fprofile-arcs -ftest-coverage"
fi
AM_CONDITIONAL([ENABLE_COVERAGE], [test x$enable_coverage = xyes])

# Checks for libraries.
m4_define(glib_required_version, 2.31.0)
m4_define(gobject_required_version, 2.18.0)

AC_CHECK_HEADERS([malloc.h])
AC_CHECK_FUNCS(mallinfo)

echo "Using JS_CFLAGS: $JS_CFLAGS"
echo "Using JS_LIBS: $JS_LIBS"

save_CFLAGS="$CFLAGS"
CFLAGS="$CFLAGS $JS_CFLAGS"
AC_MSG_CHECKING([for JS_CLASS_TRACE macro])
AC_COMPILE_IFELSE(
    [AC_LANG_PROGRAM(
        [[
            #include <jsapi.h>
	    #ifndef JS_CLASS_TRACE
	    #error "No JS_CLASS_TRACE"
	    #endif
        ]],
        [[(void) 0;]]
    )],
    [have_js_class_trace=yes],
    [have_js_class_trace=no])

if test "x$have_js_class_trace" = xyes; then
    AC_MSG_RESULT([yes])
    AC_DEFINE([HAVE_JS_CLASS_TRACE], [1], [Define if we still have JS_CLASS_TRACE])
else
    AC_MSG_RESULT([no])
fi
CFLAGS="$save_CFLAGS"

GOBJECT_INTROSPECTION_REQUIRE([1.33.2])

common_packages="glib-2.0 >= glib_required_version gobject-2.0 >= gobject_required_version mozjs185"
gjs_packages="gmodule-2.0 gthread-2.0 gobject-introspection-1.0 >= 0.10.1 $common_packages"
gjs_cairo_gobject_packages="cairo-gobject $common_packages"
gjs_cairo_packages="cairo $common_packages"
gjs_dbus_packages="dbus-glib-1 $common_packages"
gjs_gdbus_packages="gobject-2.0 >= gobject_required_version gio-2.0"
# gjs-tests links against everything
gjstests_packages="$gjstests_packages $gjs_packages"

PKG_CHECK_MODULES([GOBJECT], [gobject-2.0 >= gobject_required_version])
PKG_CHECK_MODULES([GJS], [$gjs_packages])
# Prefer cairo-gobject if we have it
PKG_CHECK_MODULES([GJS_CAIRO], [$gjs_cairo_gobject_packages], have_cairo_gobject=yes, have_cairo_gobject=no)
if test x${have_cairo_gobject} != xyes; then
  PKG_CHECK_MODULES([GJS_CAIRO], [$gjs_cairo_packages])
fi
PKG_CHECK_MODULES([GJS_DBUS], [$gjs_dbus_packages])
PKG_CHECK_MODULES([GJS_GDBUS], [$gjs_gdbus_packages])

saved_CFLAGS=$CFLAGS
CFLAGS=$GJS_DBUS_CFLAGS
saved_LIBS=$LIBS
LIBS=$GJS_DBUS_LIBS
AC_CHECK_FUNCS(dbus_message_iter_abandon_container)
CFLAGS=$saved_CFLAGS
LIBS=$saved_LIBS
PKG_CHECK_MODULES([GJSTESTS], [$gjstests_packages])

GI_DATADIR=$($PKG_CONFIG --variable=gidatadir gobject-introspection-1.0)
AC_SUBST(GI_DATADIR)

# readline
LIBS_no_readline=$LIBS

# On some systems we need to link readline to a termcap compatible
# library.
gjs_cv_lib_readline=no
AC_MSG_CHECKING([how to link readline libs])
for gjs_libtermcap in "" ncursesw ncurses curses termcap; do
  if test -z "$gjs_libtermcap"; then
    READLINE_LIBS="-lreadline"
  else
    READLINE_LIBS="-lreadline -l$gjs_libtermcap"
  fi
  LIBS="$READLINE_LIBS $LIBS_no_readline"
  AC_LINK_IFELSE(
    [AC_LANG_CALL([],[readline])],
    [gjs_cv_lib_readline=yes])
  if test $gjs_cv_lib_readline = yes; then
    break
  fi
done
if test $gjs_cv_lib_readline = no; then
  AC_MSG_RESULT([none])
  READLINE_LIBS=""
else
  AC_MSG_RESULT([$READLINE_LIBS])
  AC_DEFINE(HAVE_LIBREADLINE, 1,
    [Define if you have the readline library (-lreadline).])
fi

AC_TRY_CPP([#include <readline/readline.h>],
have_readline=yes, have_readline=no)

AM_CONDITIONAL([HAVE_READLINE], [test x$have_readline = xyes])
AC_SUBST([READLINE_LIBS])
AC_SUBST([HAVE_READLINE])

# End of readline checks: restore LIBS
LIBS=$LIBS_no_readline

AC_CHECK_FUNCS([backtrace])

dnl
dnl Tracing
dnl

AC_MSG_CHECKING([whether to include systemtap tracing support])
AC_ARG_ENABLE([systemtap],
             [AS_HELP_STRING([--enable-systemtap],
                              [Enable inclusion of systemtap trace support])],
              [ENABLE_SYSTEMTAP="${enableval}"], [ENABLE_SYSTEMTAP='no'])
AC_MSG_RESULT(${ENABLE_SYSTEMTAP})

AC_MSG_CHECKING([whether to include dtrace tracing support])
AC_ARG_ENABLE([dtrace],
             [AS_HELP_STRING([--enable-dtrace],
                              [Enable inclusion of dtrace trace support])],
              [ENABLE_DTRACE="${enableval}"], [ENABLE_DTRACE='no'])
AC_MSG_RESULT(${ENABLE_DTRACE})

AM_CONDITIONAL([ENABLE_SYSTEMTAP], [test x$ENABLE_SYSTEMTAP = xyes])
AM_CONDITIONAL([ENABLE_DTRACE], [test x$ENABLE_DTRACE = xyes -o x$ENABLE_SYSTEMTAP = xyes])

if test "x${ENABLE_DTRACE}" = xyes -o "x${ENABLE_SYSTEMTAP}" = xyes; then
  AC_CHECK_PROGS(DTRACE, dtrace)
  if test -z "$DTRACE"; then
    AC_MSG_ERROR([dtrace not found])
  fi
  AC_CHECK_HEADER([sys/sdt.h], [SDT_H_FOUND='yes'],
                  [SDT_H_FOUND='no';
                   AC_MSG_ERROR([tracing support needs sys/sdt.h header])])
  AC_DEFINE([HAVE_DTRACE], [1], [Define to 1 if using dtrace probes.])
fi

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST

gjsjsdir="\${datadir}/gjs-1.0"
gjsnativedir="\${libdir}/gjs-1.0"
AC_SUBST([gjsjsdir])
AC_SUBST([gjsnativedir])

AC_CONFIG_FILES([Makefile gjs-1.0.pc gjs-dbus-1.0.pc gjs-internals-1.0.pc])
AC_OUTPUT

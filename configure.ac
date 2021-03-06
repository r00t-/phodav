AC_PREREQ([2.59])
AC_INIT([phodav], [m4_esyscmd(build-aux/git-version-gen .tarball-version)],
                  [https://bugzilla.gnome.org/enter_bug.cgi?product=phodav])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_AUX_DIR([build-aux])
AC_CANONICAL_HOST
AC_PROG_CC

AC_CHECK_PROG([STOW], [stow], [yes], [no])
AS_IF([test "x$STOW" = "xyes" && test -d /usr/local/stow], [
  AC_MSG_NOTICE([*** Found /usr/local/stow: default install prefix set to /usr/local/stow/${PACKAGE_NAME} ***])
  ac_default_prefix="/usr/local/stow/${PACKAGE_NAME}"
])

AC_MSG_CHECKING([for native Win32])
case "$host_os" in
  *mingw*|*cygwin*)
    os_win32=yes
    gio_os=gio-windows-2.0
  ;;
  *)
    os_win32=no
    gio_os=gio-unix-2.0
  ;;
esac
AC_MSG_RESULT([$os_win32])
AM_CONDITIONAL([OS_WIN32],[test "$os_win32" = "yes"])

AM_INIT_AUTOMAKE([foreign -Wall -Wno-portability subdir-objects dist-xz])
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])
AM_MAINTAINER_MODE([enable])
LT_INIT([win32-dll])
GTK_DOC_CHECK([1.14],[--flavour no-tmpl])

gl_MANYWARN_ALL_GCC([warnings])
nw=
nw="$nw -Wsystem-headers"       # Don't let system headers trigger warnings
nw="$nw -Wundef"                # All compiler preprocessors support #if UNDEF
nw="$nw -Wtraditional"          # All compilers nowadays support ANSI C
nw="$nw -Wstrict-overflow"      # yacc/bison generated code uses that..
nw="$nw -Wconversion"           # These warnings usually don't point to mistakes.
nw="$nw -Wsign-conversion"      # Likewise.
nw="$nw -Wunused-parameter"     # Likewise.
nw="$nw -Wunused-local-typedefs" # Likewise.
nw="$nw -Wunused-macros"        # Likewise.
nw="$nw -Wunsafe-loop-optimizations" # Likewise.
nw="$nw -Wbad-function-cast"    # Likewise.
# Enable all GCC warnings not in this list.
gl_MANYWARN_COMPLEMENT([warnings], [$warnings], [$nw])
for w in $warnings; do
  gl_WARN_ADD([$w])
done
gl_WARN_ADD([-Wno-sign-compare])
gl_WARN_ADD([-Wno-unused-parameter])
gl_WARN_ADD([-Wno-missing-field-initializers])

GETTEXT_PACKAGE=phodav-2.0
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, ["$GETTEXT_PACKAGE"], [Define to the Gettext package name])
AC_SUBST(GETTEXT_PACKAGE)
IT_PROG_INTLTOOL([0.40.0])

AC_CHECK_HEADERS([attr/xattr.h])

PKG_CHECK_MODULES(GIO, [$gio_os])
PKG_CHECK_MODULES(SOUP, libsoup-2.4 >= 2.48.0 libxml-2.0)

# GLib's GUuid was introduce in GLib 2.51
# https://bugzilla.gnome.org/show_bug.cgi?id=639078
# do not build our version of it conditionally 
PKG_CHECK_EXISTS([glib-2.0 > 2.51], [build_guuid=no], [build_guuid=yes])
AM_CONDITIONAL(BUILD_GUUID, [test "x$build_guuid" = "xyes"])

AC_ARG_WITH([avahi],
  AS_HELP_STRING([--without-avahi], [Build without avahi/mdns]))

AS_IF([test "x$with_avahi" != "xno"],
  [PKG_CHECK_MODULES([AVAHI], [avahi-gobject avahi-client],
                     [have_avahi=yes], [have_avahi=no])],
  [have_avahi=no])
AS_IF([test "x$have_avahi" != "xyes"],
  [AS_IF([test "x$with_avahi" = "xyes"],
         [AC_MSG_ERROR([avahi requested but not found])])])
AS_IF([test "x$have_avahi" = "xyes"], [AC_DEFINE([WITH_AVAHI], [1], [With avahi])])

AC_ARG_ENABLE([pie],
  AS_HELP_STRING([--enable-pie=@<:@auto/yes/no@:>@],
                 [Enable position-independent-executable support @<:@default=auto@:>@]),
                 [],
                 [enable_pie="auto"])

if test "x$enable_pie" != "xno"; then
  save_CFLAGS="$CFLAGS"
  save_LDFLAGS="$LDFLAGS"
  CFLAGS="$CFLAGS -fPIE"
  LDFLAGS="$LDFLAGS -pie -Wl,-z,relro -Wl,-z,now"
  AC_MSG_CHECKING([for PIE support])
  AC_LINK_IFELSE([AC_LANG_SOURCE([int main () { return 0; }])],
                 [have_pie=yes],
                 [have_pie=no])
  AC_MSG_RESULT([$have_pie])
  if test "x$have_pie" = "xno" && test "x$enable_pie" = "xyes"; then
    AC_MSG_ERROR([pie support explicitly requested, but your toolchain does not support it])
  fi
  if test "x$have_pie" = "xyes"; then
    PIE_CFLAGS="-fPIE"
    PIE_LDFLAGS="-pie -Wl,-z,relro -Wl,-z,now"
    AC_SUBST(PIE_CFLAGS)
    AC_SUBST(PIE_LDFLAGS)
  fi
  CFLAGS="$save_CFLAGS"
  LDFLAGS="$save_LDFLAGS"
else
  have_pie=no
fi

AC_ARG_WITH([systemdsystemunitdir],
            AS_HELP_STRING([--with-systemdsystemunitdir=DIR], [Directory for systemd service files]),
            [], [with_systemdsystemunitdir=$prefix/lib/systemd/system])
AC_SUBST([SYSTEMDSYSTEMUNITDIR], [$with_systemdsystemunitdir])
AM_CONDITIONAL(HAVE_SYSTEMD, [test -n "$with_systemdsystemunitdir"])

AC_ARG_WITH([udevdir],
            AS_HELP_STRING([--with-udevdir=DIR], [Directory for udev files]),
            [], [with_udevdir=$prefix/lib/udev])
AC_SUBST([UDEVDIR], [$with_udevdir])
AM_CONDITIONAL(HAVE_UDEV, [test -n "$with_udevdir"])

AC_ARG_VAR([XMLTO], [Path to xmlto command])
AC_PATH_PROG([XMLTO], [xmlto])
AC_ARG_VAR([ASCIIDOC], [Path to asciidoc command])
AC_PATH_PROG([ASCIIDOC], [asciidoc])
AM_CONDITIONAL([HAVE_DOCTOOLS], [test "x$XMLTO" != "x" && test "x$ASCIIDOC" != "x"])
if test "x$XMLTO" = "x" || test "x$ASCIIDOC" = "x"; then
  AC_MSG_WARN([xmlto or asciidoc not found - cannot create man pages without it])
fi

AC_ARG_WITH([buildid],
AS_HELP_STRING([--with-buildid=id], [Set additional build version details]))
AC_DEFINE_UNQUOTED([BUILDID], "$with_buildid", [Build version details])
if test "x$with_buildid" != x; then
  AC_SUBST([BUILDID], "-$with_buildid")
fi

major=`echo $PACKAGE_VERSION | cut -d. -f1`
minor=`echo $PACKAGE_VERSION | cut -d. -f2`
micro=`echo $PACKAGE_VERSION | cut -d. -f3 | cut -d- -f1`
buildid=`echo $with_buildid | cut -d. -f1`
if test "x$buildid" = x; then
  buildid=0
fi
build=`expr $micro \* 256 + $buildid`
WINDOWS_PRODUCTVERSION="$major.$minor.$build"
AC_SUBST([WINDOWS_PRODUCTVERSION])

case "$host" in
  amd64*|x86_64*)
    WIXL_ARCH="x64"
  ;;
  *)
    WIXL_ARCH="x86"
  ;;
esac
AC_SUBST(WIXL_ARCH)

AC_CONFIG_FILES([
Makefile
chezdav.wxs
libphodav-2.0.pc
doc/Makefile
doc/reference/Makefile
doc/reference/phodav-docs.sgml
libphodav/Makefile
po/Makefile.in
spice-webdavd.wxs
spice/Makefile
])
AC_OUTPUT

dnl ==========================================================================
AC_MSG_NOTICE([

    $PACKAGE $VERSION
    =================

      prefix:                   $prefix
      c compiler:               $CC

      Avahi:                    $have_avahi
])

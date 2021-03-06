
AC_INIT([xtables-addons], [1.22])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])
AC_PROG_INSTALL
AM_INIT_AUTOMAKE([1.10 -Wall foreign subdir-objects])
AC_PROG_CC
AM_PROG_CC_C_O
AC_DISABLE_STATIC
AC_PROG_LIBTOOL

AC_ARG_WITH([kbuild],
	AS_HELP_STRING([--with-kbuild=PATH],
	[Path to kernel build directory [[/lib/modules/CURRENT/build]]]),
	[kbuilddir="$withval"],
	[kbuilddir="/lib/modules/$(uname -r)/build"])
#
# check for --without-kbuild
#
if [[ "$kbuilddir" == no ]]; then
	kbuilddir="";
fi

AC_ARG_WITH([ksource],,[ksourcedir="$withval"])
AC_ARG_WITH([xtables],
	AS_HELP_STRING([--with-xtables=PATH],
	[Path to the Xtables includes [[none]]]),
	[xtables_location="$withval"])
AC_ARG_WITH([xtlibdir],
	AS_HELP_STRING([--with-xtlibdir=PATH],
	[Path where to install Xtables extensions [[LIBEXECDIR/xtables]]]),
	[xtlibdir="$withval"],
	[xtlibdir='${libexecdir}/xtables'])

#
# --with-xtables= overrides a possibly installed pkgconfig file.
#
if [[ -n "$xtables_location" ]]; then
	AC_MSG_CHECKING([xtables.h presence])
	if [[ -f "$xtables_location/xtables.h" ]]; then
		AC_MSG_RESULT([$xtables_location/xtables.h])
		libxtables_CFLAGS="-I $xtables_location";
	elif [[ -f "$xtables_location/include/xtables.h" ]]; then
		AC_MSG_RESULT([$xtables_location/include/xtables.h])
		libxtables_CFLAGS="-I $xtables_location/include";
	fi;
	if [[ -z "$xtables_CFLAGS" ]]; then
		if [[ -f "$includedir/xtables.h" ]]; then
			AC_MSG_RESULT([$includedir/xtables.h])
		else
			AC_MSG_RESULT([no])
		fi;
	fi;
	libxtables_LIBS="-lxtables";
	AC_SUBST([libxtables_CFLAGS])
	AC_SUBST([libxtables_LIBS])
else
	PKG_CHECK_MODULES([libxtables], [xtables >= 1.4.3])
fi;
AC_CHECK_HEADERS([linux/netfilter/x_tables.h], [],
	[AC_MSG_ERROR([You need to have linux/netfilter/x_tables.h, either through /usr/include or the iptables directory (--with-xtables=)])])

regular_CFLAGS="-D_LARGEFILE_SOURCE=1 -D_LARGE_FILES -D_FILE_OFFSET_BITS=64 \
	-D_REENTRANT -Wall -Waggregate-return -Wmissing-declarations \
	-Wmissing-prototypes -Wredundant-decls -Wshadow -Wstrict-prototypes \
	-Winline -pipe -DXTABLES_LIBDIR=\\\"\${xtlibdir}\\\"";
kinclude_CFLAGS="";
if [[ -n "$kbuilddir" ]]; then
	kinclude_CFLAGS="$kinclude_CFLAGS -I $kbuilddir/include";
fi;
if [[ -n "$ksourcedir" ]]; then
	kinclude_CFLAGS="$kinclude_CFLAGS -I $ksourcedir/include";
fi;

#
# check kernel version
#
if grep -q "CentOS release 5\." /etc/redhat-release 2>/dev/null ||
    grep -q "Red Hat Enterprise Linux Server release 5" /etc/redhat-release 2>/dev/null; then
	# しまった!
	# Well, just a warning. Maybe the admin updated the kernel.
	echo "WARNING: This distribution's shipped kernel is not supported.";
fi;
krel="$(make -sC ${kbuilddir} kernelrelease)";
krel="${krel%%-*}";
kmajor="${krel%%.*}";
krel="${krel#*.}";
kminor="${krel%%.*}";
krel="${krel#*.}";
kmicro="${krel%%.*}";
if test "$kmicro" = "$krel"; then
	kstable=0;
else
	kstable="${krel#*.}";
	if test -z "$kstable"; then
		kstable=0;
	fi;
fi;
echo "Found kernel version $kmajor.$kminor.$kmicro.$kstable in $kbuilddir";
if test "$kmajor" -gt 2 -o "$kminor" -gt 6 -o "$kmicro" -gt 32; then
	echo "WARNING: You are trying a newer kernel. Results may vary. :-)";
elif test \( "$kmajor" -lt 2 -o "$kminor" -lt 6 -o "$kmicro" -lt 17 \) -o \
    \( "$kmajor" -eq 2 -a "$kminor" -eq 6 -a "$kmicro" -eq 18 -a \
    "$kstable" -lt 5 \); then
	echo "ERROR: That kernel version is not supported. Please see INSTALL for minimum configuration.";
	exit 1;
fi;

AC_SUBST([regular_CFLAGS])
AC_SUBST([kinclude_CFLAGS])
AC_SUBST([kbuilddir])
AC_SUBST([ksourcedir])
AC_SUBST([xtlibdir])
AC_CONFIG_FILES([Makefile Makefile.iptrules Makefile.mans
	extensions/Makefile extensions/ipset/Makefile])
AC_OUTPUT

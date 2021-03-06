AC_PREREQ(2.59)
AC_INIT(geoclue, 0.12.99, http://geoclue.freedesktop.org)

AC_CONFIG_SRCDIR(geoclue/gc-iface-geoclue.c)
AM_CONFIG_HEADER(config.h)
AM_INIT_AUTOMAKE([1.9 foreign])
GEOCLUE_VERSION=0.12.0

# Enable silent build when available (Automake 1.11)
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

AC_PROG_CC
AC_ISC_POSIX

AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_PROG_GCC_TRADITIONAL
AM_PROG_LIBTOOL
GLIB_GSETTINGS

#GTK_DOC_CHECK(1.0)
AC_CHECK_PROGS(XSLT, xsltproc)

AC_ARG_ENABLE(system-bus,
	      [AC_HELP_STRING([--enable-system-bus],
			      [Use the system bus instead of session bus])],
	      enable_system_bus="$enableval",
	      enable_system_bus=no)

AM_CONDITIONAL(USE_SYSTEM_BUS, test x$enable_system_bus = xyes)
if test x$enable_system_bus = xyes; then
	AC_DEFINE(GEOCLUE_DBUS_BUS, DBUS_BUS_SYSTEM, Use the system bus)
else
	AC_DEFINE(GEOCLUE_DBUS_BUS, DBUS_BUS_SESSION, Use the session bus)
fi

AC_ARG_ENABLE(tests,[  --disable-tests           disable test libraries ], enable_tests=$enableval,enable_tests=yes)
if test "x$enable_tests" = "xyes"; then
   BUILD_TESTS=test
fi
AC_SUBST(BUILD_TESTS)

PKG_CHECK_MODULES(GEOCLUE, [
		  glib-2.0
		  gobject-2.0
		  dbus-glib-1 >= 0.86
		  libxml-2.0
])
AC_SUBST(GEOCLUE_LIBS)
AC_SUBST(GEOCLUE_CFLAGS)

PKG_CHECK_MODULES(MASTER, [
		  gio-2.0 >= 2.25.7
		  glib-2.0
])
AC_SUBST(MASTER_LIBS)
AC_SUBST(MASTER_CFLAGS)

PKG_CHECK_MODULES([DROIDHEADERS], [android-headers])
AC_SUBST(DROIDHEADERS_CFLAGS)

PKG_CHECK_MODULES([HYBRIS], [libhardware])
AC_SUBST(HYBRIS_CFLAGS)
AC_SUBST(HYBRIS_LIBS)

AC_PATH_PROG(DBUS_BINDING_TOOL, dbus-binding-tool)
AC_PATH_PROG(GLIB_GENMARSHAL, glib-genmarshal)

DBUS_SERVICES_DIR="${datadir}/dbus-1/services"
AC_SUBST(DBUS_SERVICES_DIR)
AC_DEFINE_UNQUOTED(DBUS_SERVICES_DIR, "$DBUS_SERVICES_DIR", [Where services dir for D-Bus is])

CFLAGS="$CFLAGS -g -Wall -Wno-format"

# -----------------------------------------------------------
# gtk+
# -----------------------------------------------------------
AC_ARG_ENABLE(gtk,
              AS_HELP_STRING([--enable-gtk=@<:@no/yes/auto@:>@],
                             [build with gtk support]), ,
                             enable_gtk=auto)
 
if test "x$enable_gtk" != "xno"; then
   PKG_CHECK_MODULES(GTK,
   [
      gtk+-2.0
   ], have_gtk="yes", have_gtk="no")
 
   if test "x$have_gtk" = "xyes"; then
      AC_DEFINE(HAVE_GTK, 1, [Define if you have gtk+])
   fi
else
   have_gtk=no
fi

if test "x$enable_gtk" = "xyes" -a "x$have_gtk" != "xyes"; then
   AC_MSG_ERROR([Couldn't find gtk dependencies.])
fi

AM_CONDITIONAL(HAVE_GTK, test "x$have_gtk" = "xyes")
AC_SUBST(GTK_LIBS)
AC_SUBST(GTK_CFLAGS)

# -----------------------------------------------------------
# connectivity
# -----------------------------------------------------------

CONNECTIVITY="None"

AC_ARG_ENABLE(conic,
              AS_HELP_STRING([--enable-conic=@<:@no/yes/auto@:>@],
                             [build with conic support]), ,
                             enable_conic=auto)

if test "x$enable_conic" != "xno"; then
   PKG_CHECK_MODULES(CONIC,
   [
      conic
   ], have_conic="yes", have_conic="no")

   if test "x$have_conic" = "xyes"; then
      CONNECTIVITY="Maemo LibConIC"
      CONNECTIVITY_LIBS=${CONIC_LIBS}
      CONNECTIVITY_CFLAGS=${CONIC_CFLAGS}
      AC_DEFINE(HAVE_CONIC, 1, [define if libconic is installed])
   fi
else
   have_conic=no
fi

if test "x$enable_conic" = "xyes" -a "x$have_conic" != "xyes"; then
   AC_MSG_ERROR([Couldn't find conic dependencies.])
fi

AC_ARG_ENABLE(connman,
              AS_HELP_STRING([--enable-connman=@<:@no/yes/auto@:>@],
                             [build with connman support]), ,
                             enable_connman=auto)

if test "x$enable_connman" != "xno"; then
    CONNECTIVITY="ConnMan"
    AC_DEFINE(HAVE_CONNMAN, 1, [define if libconmann is installed])
else
   have_connman=no
fi

AC_ARG_ENABLE(networkmanager,
              AS_HELP_STRING([--enable-networkmanager=@<:@no/yes/auto@:>@],
                             [build with NetworkManager support]), ,
                             enable_networkmanager=auto)

if test "x$enable_networkmanager" != "xno"; then
   PKG_CHECK_MODULES(NETWORK_MANAGER,
   [
      NetworkManager libnm-glib libnm-util
   ], have_networkmanager="yes", have_networkmanager="no")

   if test "x$have_networkmanager" = "xyes"; then
      CONNECTIVITY="Network Manager"
      CONNECTIVITY_LIBS=${NETWORK_MANAGER_LIBS}
      CONNECTIVITY_CFLAGS=${NETWORK_MANAGER_CFLAGS}
      AC_DEFINE(HAVE_NETWORK_MANAGER, 1, [define if Network Manager is installed])
   fi
else
   have_networkmanager=no
fi

if test "x$enable_networkmanager" = "xyes" -a "x$have_networkmanager" != "xyes"; then
   AC_MSG_ERROR([Couldn't find Network Manager dependencies.])
fi

AC_SUBST(CONNECTIVITY_LIBS)
AC_SUBST(CONNECTIVITY_CFLAGS)

PROVIDER_SUBDIRS="example hostip hybris geonames nominatim manual plazes localnet yahoo gsmloc"

# -----------------------------------------------------------
# gypsy / gpsd / skyhook
# -----------------------------------------------------------

GYPSY_REQUIRED=0.7.1
AC_ARG_ENABLE(gypsy,
              AS_HELP_STRING([--enable-gypsy=@<:@no/yes/auto@:>@],
                             [build with gypsy support]), ,
                             enable_gypsy=auto)

if test "x$enable_gypsy" != "xno"; then
   PKG_CHECK_MODULES(GYPSY,
   [
      gypsy >= $GYPSY_REQUIRED
   ], have_gypsy="yes", have_gypsy="no")

   if test "x$have_gypsy" = "xyes"; then
      PROVIDER_SUBDIRS="$PROVIDER_SUBDIRS gypsy"
   else
      NO_BUILD_PROVIDERS="$NO_BUILD_PROVIDERS gypsy"
   fi
 else
   have_gypsy=no
fi

if test "x$enable_gypsy" = "xyes" -a "x$have_gypsy" != "xyes"; then
   AC_MSG_ERROR([Couldn't find gypsy dependencies.])
fi
AC_SUBST(GYPSY_LIBS)
AC_SUBST(GYPSY_CFLAGS)

AC_ARG_ENABLE(gpsd,
              AS_HELP_STRING([--enable-gpsd=@<:@no/yes/auto@:>@],
                             [build with gpsd support]), ,
                             enable_gpsd=auto)

if test "x$enable_gpsd" != "xno"; then
   PKG_CHECK_MODULES(GPSD, [libgps >= 2.91],  have_gpsd="yes", have_gpsd="no")
   if test "x$have_gpsd" = "xyes"; then
      PROVIDER_SUBDIRS="$PROVIDER_SUBDIRS gpsd"
   else
      NO_BUILD_PROVIDERS="$NO_BUILD_PROVIDERS gpsd"
   fi
else
   have_gpsd=no
fi

if test "x$enable_gpsd" = "xyes" -a "x$have_gpsd" != "xyes"; then
   AC_MSG_ERROR([Couldn't find gpsd dependencies - libgps >= 2.90.])
fi
AC_SUBST(GPSD_LIBS)
AC_SUBST(GPSD_CFLAGS)

AC_ARG_ENABLE(skyhook,
              AS_HELP_STRING([--enable-skyhook=@<:@no/yes/auto@:>@],
                             [build with skyhook support]), ,
                             enable_skyhook=auto)

if test "x$enable_skyhook" != "xno"; then
    PKG_CHECK_MODULES(SKYHOOK, [
		  libsoup-gnome-2.4
    ], have_skyhook=yes, have_skyhook=no)

   if test "x$have_skyhook" = "xyes"; then
      PROVIDER_SUBDIRS="$PROVIDER_SUBDIRS skyhook"
      GPSD_LIBS="-lgps"
   else
      NO_BUILD_PROVIDERS="$NO_BUILD_PROVIDERS skyhook"
   fi
 else
   have_skyhook=no
fi

if test "x$enable_skyhook" = "xyes" -a "x$have_skyhook" != "xyes"; then
   AC_MSG_ERROR([Couldn't find gpsd dependencies.])
fi
AC_SUBST(SKYHOOK_LIBS)
AC_SUBST(SKYHOOK_CFLAGS)

AC_SUBST(PROVIDER_SUBDIRS)
AC_SUBST(NO_BUILD_PROVIDERS)

AC_CONFIG_FILES([
geoclue.pc
Makefile
docs/Makefile
docs/reference/Makefile
docs/tools/Makefile
interfaces/Makefile
geoclue/Makefile
example/Makefile
test/Makefile
providers/Makefile
providers/example/Makefile
providers/gypsy/Makefile
providers/gpsd/Makefile
providers/hostip/Makefile
providers/hybris/Makefile
providers/geonames/Makefile
providers/manual/Makefile
providers/nominatim/Makefile
providers/plazes/Makefile
providers/localnet/Makefile
providers/yahoo/Makefile
providers/gsmloc/Makefile
providers/skyhook/Makefile
src/Makefile
])

AC_OUTPUT

echo ""
echo "Geoclue ${VERSION} has been configured as follows: "
echo "---------------------------------------------------"
echo "Source code location:   ${srcdir}"
echo "Compiler:               ${CC}"
echo "Network connectivity:   ${CONNECTIVITY}"
echo "Providers:              ${PROVIDER_SUBDIRS}"
echo "Excluded providers:    ${NO_BUILD_PROVIDERS}"
echo ""

dnl Process this file with autoconf to produce a configure script.

AC_INIT([forked-daapd], [21.0])
AC_CONFIG_SRCDIR([config.h.in])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_HEADERS([config.h])
AM_INIT_AUTOMAKE([-Wno-portability subdir-objects])

AC_USE_SYSTEM_EXTENSIONS

dnl Checks for programs.
AC_PROG_CC
AM_PROG_CC_C_O
LT_INIT([disable-static])

AC_CHECK_PROG(GPERF, [gperf], [gperf])
if test "x$GPERF" = x; then
	AC_MSG_ERROR([GNU gperf not found, please install it])
fi
AC_SUBST(GPERF)

AC_CHECK_PROG(ANTLR, [antlr3], [antlr3])
if test "x$ANTLR" = x; then
	if test -d $srcdir/src/pregen; then
		for f in $srcdir/src/pregen/*; do
			bf=`basename $f`
			ln -sf pregen/$bf $srcdir/src/$bf
		done
		AC_MSG_NOTICE([antlr3 wrapper not found, using pre-generated files])
	else
		AC_MSG_ERROR([antlr3 wrapper not found and pre-generated files not available])
	fi
fi
AC_SUBST(ANTLR)
AM_CONDITIONAL(COND_ANTLR, test "x$ANTLR" != x)

CFLAGS="$CFLAGS -Wall -D_LARGEFILE_SOURCE"

AC_CHECK_HEADERS([sys/wait.h])
AC_CHECK_HEADERS([sys/param.h])
AC_CHECK_HEADERS([sys/select.h])
AC_CHECK_HEADERS([dirent.h])
AC_CHECK_FUNCS(posix_fadvise)
AC_CHECK_FUNCS(strptime)
AC_CHECK_FUNCS(strtok_r)
AC_CHECK_FUNCS(timegm)
AC_CHECK_FUNCS(euidaccess)

dnl Large File Support (LFS)
AC_SYS_LARGEFILE
AC_TYPE_OFF_T

AC_ARG_ENABLE(flac, AS_HELP_STRING([--enable-flac], [extract FLAC metadata with libflac (default=no)]),
                      use_flac=true;
                      CPPFLAGS="${CPPFLAGS} -DFLAC")

AC_ARG_ENABLE(musepack, AS_HELP_STRING([--enable-musepack], [extract Musepack metadata with taglib (default=no)]),
                      use_musepack=true;
                      CPPFLAGS="${CPPFLAGS} -DMUSEPACK")

AC_ARG_ENABLE(itunes, AS_HELP_STRING([--enable-itunes], [enable iTunes library support (default=no)]),
                      use_itunes=true;
                      CPPFLAGS="${CPPFLAGS} -DITUNES")

AC_ARG_ENABLE(spotify, AS_HELP_STRING([--enable-spotify], [enable Spotify library support (default=no)]),
                      use_spotify=true;
                      CPPFLAGS="${CPPFLAGS} -DSPOTIFY")

AC_ARG_ENABLE(lastfm, AS_HELP_STRING([--enable-lastfm], [enable LastFM support (default=no)]),
                      use_lastfm=true;
                      CPPFLAGS="${CPPFLAGS} -DLASTFM")

AM_CONDITIONAL(COND_FLAC, test x$use_flac = xtrue)
AM_CONDITIONAL(COND_MUSEPACK, test x$use_musepack = xtrue)
AM_CONDITIONAL(COND_ITUNES, test x$use_itunes = xtrue)
AM_CONDITIONAL(COND_SPOTIFY, test x$use_spotify = xtrue)
AM_CONDITIONAL(COND_LASTFM, test x$use_lastfm = xtrue)

AC_ARG_WITH(oss4, AS_HELP_STRING([--with-oss4=includedir], [use OSS4 with soundcard.h in includedir (default /usr/lib/oss/include/sys)]),
  [ case "$withval" in
    no)
    case "$host" in
      *-*-linux-*)
      use_oss4=false
      ;;
      *-*-kfreebsd*-*|*-*-freebsd-*)
      AC_MSG_ERROR([OSS4 must be enabled on FreeBSD systems])
      ;;
    esac
    ;;
    yes)
    use_oss4=true
    oss4_includedir=/usr/lib/oss/include/sys
    ;;
    [[\\/$]]* | ?:[[\\/]]* )
    use_oss4=true
    oss4_includedir="$withval"
    ;;
    *)
    AC_MSG_ERROR([expected an absolute directory name for --with-oss4: $withval])
    ;;
    esac ],
  [ case "$host" in
    *-*-linux-*)
    use_oss4=false
    ;;
    *-*-kfreebsd*-*|*-*-freebsd-*)
    use_oss4=true
    oss4_includedir=/usr/lib/oss/include/sys
    ;;
    esac ])

dnl Checks for libraries.
gl_LIBUNISTRING

if test x$HAVE_LIBUNISTRING != xyes; then
   AC_MSG_ERROR([GNU libunistring is required])
fi

PKG_CHECK_MODULES(ZLIB, [ zlib ])
PKG_CHECK_MODULES(CONFUSE, [ libconfuse ])
PKG_CHECK_MODULES(AVAHI, [ avahi-client >= 0.6.24 ])
PKG_CHECK_MODULES(SQLITE3, [ sqlite3 >= 3.5.0 ])

save_LIBS="$LIBS"
LIBS="$SQLITE3_LIBS"
dnl Check that SQLite3 has the unlock notify API built-in
AC_CHECK_LIB([sqlite3], [sqlite3_unlock_notify], [], AC_MSG_ERROR([SQLite3 was built without unlock notify support]))
dnl Check that SQLite3 has been built with threadsafe operations
AC_MSG_CHECKING([if SQLite3 was built with threadsafe operations support])
AC_LANG_PUSH([C])
AC_RUN_IFELSE(
  [AC_LANG_PROGRAM([dnl
    #include <sqlite3.h>
    ], [dnl
    int ret = sqlite3_config(SQLITE_CONFIG_MULTITHREAD);
    if (ret != SQLITE_OK)
      return 1;
    return 0;])],
  [AC_MSG_RESULT([yes])], [AC_MSG_ERROR([SQLite3 was not built with threadsafe operations support])],
  [AC_MSG_RESULT([runtime will tell])])
AC_LANG_POP([C])
LIBS="$save_LIBS"

PKG_CHECK_EXISTS([ libavcodec >= 54.35 ], [ 
  PKG_CHECK_MODULES(LIBAV, [ libavformat libavcodec libswscale libavresample libavutil ]) 
], [ 
  PKG_CHECK_MODULES(LIBAV, [ libavformat libavcodec libswscale libavutil ])
])

save_LIBS="$LIBS"
dnl Check for av_lock_manager (ffmpeg >= 0.5.1)
AC_CHECK_LIB([avcodec], [av_lockmgr_register], , AC_MSG_ERROR([libav (ffmpeg) >= 0.5.1 required]))
dnl Check for avcodec_find_best_pix_fmt_of_list (is only in ffmpeg, not libav)
AC_CHECK_LIB([avcodec], [avcodec_find_best_pix_fmt_of_list],
	AC_DEFINE(FFMPEG_INCOMPATIBLE_API, 1, [Define to 1 if you have ffmpeg (and not libav).]))
LIBS="$save_LIBS"

# Check for libavformat >= 53; url -> avio switch
_PKG_CONFIG([libavformat_VERSION], [atleast-version=53], [libavformat])
if test $pkg_failed = yes; then
	AM_CONDITIONAL(COND_AVIO, false)
else
	AM_CONDITIONAL(COND_AVIO, true)
fi

PKG_CHECK_MODULES(MINIXML, [ mxml ])

PKG_CHECK_MODULES(LIBEVENT, [ libevent >= 2 ],
        AC_DEFINE(HAVE_LIBEVENT2, 1, [Define to 1 if you have libevent 2]),
	try_libevent1=true;
)

AM_CONDITIONAL(COND_LIBEVENT1, false)
AM_CONDITIONAL(COND_LIBEVENT20, false)
AM_CONDITIONAL(COND_LIBEVENT21, false)

if test x$try_libevent1 = xtrue; then
        AC_CHECK_HEADER(event.h, , AC_MSG_ERROR([event.h not found]))
        AC_CHECK_LIB([event_core], [event_init], [LIBEVENT_LIBS="-levent_core"], AC_MSG_ERROR([libevent not found]))
	AC_SUBST(LIBEVENT_LIBS)
	AM_CONDITIONAL(COND_LIBEVENT1, true)
else
	PKG_CHECK_EXISTS([ libevent >= 2.1 ],
		AM_CONDITIONAL(COND_LIBEVENT21, true),
		AM_CONDITIONAL(COND_LIBEVENT20, true)
	)
	PKG_CHECK_EXISTS([ libevent >= 2.1.4 ], ,
		AC_DEFINE(HAVE_LIBEVENT2_OLD, 1, [Define to 1 if you have libevent 2 (<2.1.4)])
	)
fi

AC_CHECK_HEADER(avl.h, , AC_MSG_ERROR([avl.h not found]))
AC_CHECK_LIB([avl], [avl_alloc_tree], [LIBAVL_LIBS="-lavl"], AC_MSG_ERROR([libavl not found]))
AC_SUBST(LIBAVL_LIBS)

AC_CHECK_HEADER(antlr3.h, , AC_MSG_ERROR([antlr3.h not found]))
AC_CHECK_LIB([antlr3c], [antlr3BaseRecognizerNew], [ANTLR3C_LIBS="-lantlr3c"], AC_MSG_ERROR([ANTLR3 C runtime (libantlr3c) not found]))
AC_CHECK_LIB([antlr3c], [antlr3NewAsciiStringInPlaceStream],
	AC_DEFINE(ANTLR3C_NEW_INPUT, 0, [define if antlr3 C runtime uses new input routines]),
	AC_DEFINE(ANTLR3C_NEW_INPUT, 1, [define if antlr3 C runtime uses new input routines]))
AC_SUBST(ANTLR3C_LIBS)

AM_PATH_LIBGCRYPT([1:1.2.0], , AC_MSG_ERROR([libgcrypt not found]))
AM_PATH_GPG_ERROR([1.6], , AC_MSG_ERROR([libgpg-error not found]))

if test x$use_flac = xtrue; then
	PKG_CHECK_MODULES(FLAC, [ flac ])
fi

if test x$use_musepack = xtrue; then
	PKG_CHECK_MODULES(TAGLIB, [ taglib_c ])
fi

if test x$use_itunes = xtrue; then
	PKG_CHECK_MODULES(LIBPLIST, [ libplist >= 0.16 ])
fi

if test x$use_spotify = xtrue; then
	AC_CHECK_HEADER(libspotify/api.h, , AC_MSG_ERROR([libspotify/api.h not found]))
	AC_DEFINE(HAVE_SPOTIFY_H, 1, [Define to 1 if you have the <libspotify/api.h> header file.])
	dnl Don't link to libspotify, but instead enable dynamic linking
	SPOTIFY_CFLAGS="-rdynamic"
	SPOTIFY_LIBS="-ldl"
	AC_SUBST(SPOTIFY_CFLAGS)
	AC_SUBST(SPOTIFY_LIBS)
fi

if test x$use_lastfm = xtrue; then
	PKG_CHECK_MODULES(LIBCURL, [ libcurl ])
	AC_CHECK_LIB([mxml], [mxmlGetOpaque],
		AC_DEFINE(HAVE_MXML_GETOPAQUE, 1, [Define to 1 if your mxml has mxmlGetOpaque.]))
fi

case "$host" in
  *-*-linux-*)
    if test x$use_oss4 != xtrue; then
        use_alsa=true
	PKG_CHECK_MODULES(ALSA, [ alsa ])
	AC_DEFINE(LAUDIO_USE_ALSA, 1, [define if local audio output uses ALSA])
    fi

    AC_CHECK_HEADERS([sys/eventfd.h])
    AC_CHECK_FUNCS(eventfd_write)

    AC_CHECK_HEADER(sys/signalfd.h, , AC_MSG_ERROR([signalfd required; glibc 2.9+ recommended]))

    AC_CHECK_HEADER(sys/timerfd.h, , AC_MSG_ERROR([timerfd required; glibc 2.8+ recommended]))
    AC_CHECK_FUNC(timerfd_create, , AC_MSG_ERROR([timerfd required; glibc 2.8+ recommended]))
    ;;
esac

if test x$use_oss4 = xtrue; then
   OSS4CPPFLAGS="-I$oss4_includedir"

   save_CPPFLAGS="$CPPFLAGS"
   CPPFLAGS="$OSS4CPPFLAGS"

   AC_CHECK_HEADER(soundcard.h, , AC_MSG_ERROR([soundcard.h not found in $oss4_includedir]))

   CPPFLAGS="$save_CPPFLAGS"
fi

AM_CONDITIONAL(COND_ALSA, test x$use_alsa = xtrue)
AM_CONDITIONAL(COND_OSS4, test x$use_oss4 = xtrue)
AC_SUBST(OSS4CPPFLAGS)

AC_CHECK_SIZEOF(void *)

AC_CHECK_HEADERS(getopt.h,,)
AC_CHECK_HEADERS(stdint.h,,)

dnl Checks for header files.
AC_HEADER_STDC
AC_HEADER_SYS_WAIT

AC_OUTPUT(src/Makefile sqlext/Makefile Makefile)
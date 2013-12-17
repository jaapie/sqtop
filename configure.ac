AC_PREREQ([2.64])
AC_INIT([sqtop], [2013-12-17], [o.palij@gmail.com])

AC_CONFIG_AUX_DIR(cfgaux)
AC_CONFIG_MACRO_DIR([cfgaux])

AC_CONFIG_SRCDIR([src/sqtop.cpp])
AC_CONFIG_HEADERS([src/config.h])

AM_INIT_AUTOMAKE([foreign])

AC_CANONICAL_HOST

AC_GNU_SOURCE

AC_ARG_ENABLE(ui,
[  --enable-ui     build with ncurses user interface],
[case "${enableval}" in
  yes) enable_ui=true ;;
  no)  enable_ui=false ;;
  *) AC_MSG_ERROR(bad value ${enableval} for --enable-ui) ;;
esac],[enable_ui=true])
if test x$enable_ui = xtrue; then
  AC_DEFINE(ENABLE_UI, 1, [build with ncurses user interface])
fi
AM_CONDITIONAL(ENABLE_UI, test x$enable_ui = xtrue)

AC_ARG_WITH(resolver,
[  --with-resolver=TYPE     technique sqtop should use for name resolution.
       netdb               use gethostbyaddr_r in multiple threads
       netdb_getnameinfo   use getnameinfo (guess single or multiple threads)
       netdb_gethostbyaddr use gethostbyaddr in single thread
       guess               try to guess reasonable value (default).],
    [resolver=$withval],
    [resolver=guess])

if test x$resolver = xyes; then
   resolver=guess
fi

AC_LANG([C++])
AC_PROG_CXX
AC_PROG_INSTALL
AX_CXX_COMPILE_STDCXX_11([noext], [], [AC_MSG_ERROR([Unable to find cpp11 compatible compiler]); break;])

# Checks for header files.
AC_CHECK_HEADERS([arpa/inet.h netdb.h netinet/in.h getopt.h sys/socket.h], [], [AC_MSG_ERROR([Unable to find some C headers]); break;])
AC_CHECK_HEADERS([sstream iomanip algorithm iostream string vector map cstdlib cstring cerrno climits], [], [AC_MSG_ERROR([Unable to find some C++ headers]); break;])

# Checks for library functions.
AC_CHECK_FUNCS([gethostbyname memset socket strerror], [], [AC_MSG_ERROR([Unable to find some C functions]); break;])

if test x$enable_ui = xtrue; then
   AC_CHECK_HEADERS([ncurses.h], [], [AC_MSG_ERROR([Unable to find the ncurses headers]); break;])
   AC_CHECK_HEADERS([pthread.h csignal], [], [AC_MSG_ERROR([Unable to find the threading headers]); break;])
   AC_CHECK_LIB([ncurses], [initscr], [], [AC_MSG_ERROR([Unable to find the ncurses library]); break;])
   AC_CHECK_LIB([pthread], [pthread_create], [], [AC_MSG_ERROR([Unable to find the pthread library]); break;])
   AC_CHECK_FUNCS([pthread_mutex_lock pthread_mutexattr_init pthread_mutexattr_settype pthread_mutex_init pthread_mutexattr_destroy pthread_mutex_destroy pthread_mutex_unlock pthread_create pthread_cancel], [], [AC_MSG_ERROR([Unable to find some pthread functions]); break;])
   AC_CHECK_FUNCS([clear refresh endwin mvaddstr mvchgat mvhline getch beep initscr nonl keypad noecho cbreak halfdelay addnstr], [], [AC_MSG_ERROR([Unable to find some ncurses functions]); break;])
fi

if test x$resolver != xno; then
   specified_resolver=$resolver
   if test x$resolver = xguess; then
      resolver=netdb
   fi

   if test x$resolver = xnetdb ; then
      AC_SEARCH_LIBS(gethostbyaddr_r, [nsl], , [resolver=guess])
      if test x$resolver = xguess && test x$specified_resolver != xguess ; then
         dnl They wanted gethostbyaddr_r, but they can't have it, so stop.
         AC_MSG_ERROR([no library defines gethostbyaddr_r])
      fi
   fi

   if test x$resolver = xnetdb ; then
       AC_DEFINE(USE_GETHOSTBYADDR_R, 1, [use gethostbyaddr_r for name resolution])
   fi

   if test x$resolver = xguess ; then
      resolver=netdb_getnameinfo
   fi

   if test x$resolver = xnetdb_getnameinfo ; then
      AC_SEARCH_LIBS(getnameinfo, [nsl], , [resolver=guess])
      if test x$resolver = xguess && test x$specified_resolver != xguess ; then
         dnl They wanted getnameinfo, but they can't have it, so stop.
         AC_MSG_ERROR([no library defines getnameinfo])
      fi
   fi

   if test x$resolver = xnetdb_getnameinfo ; then
      AC_DEFINE(USE_GETNAMEINFO, 1, [use getnameinfo for name resolution])

      case "$host_os" in
         darwin10.[[0-4]]*) threadsafe_getnameinfo=0 ;;
         darwin1[[0-1]].*) threadsafe_getnameinfo=1 ;;
         linux*) threadsafe_getnameinfo=1 ;;
         *) threadsafe_getnameinfo=0 ;;
      esac
      if test $threadsafe_getnameinfo = 1; then
         AC_DEFINE(THREADSAFE_GETNAMEINFO, 1, [resolving via getnameinfo can be done in multiple threads])
      fi
   fi

   if test x$resolver = xguess ; then
      resolver=netdb_gethostbyaddr
   fi

   if test x$resolver = xnetdb_gethostbyaddr; then
      AC_SEARCH_LIBS(gethostbyaddr, [nsl], , [resolver=guess])
      if test x$resolver = xguess && test x$specified_resolver != xguess ; then
         dnl They wanted gethostbyaddr, but they can't have it, so stop.
         AC_MSG_ERROR([no library defines gethostbyaddr])
      fi
   fi

   if test x$resolver = xnetdb_gethostbyaddr; then
      AC_DEFINE(USE_GETHOSTBYADDR, 1, [use gethostbyaddr for name resolution])
   fi

   if test x$resolver = xguess; then
      AC_MSG_NOTICE([failed to find reasonable resolver, name resolution will not be available!!!])
   else
      AC_DEFINE(WITH_RESOLVER, 1, [use name resolution])
   fi
fi
AM_CONDITIONAL(WITH_RESOLVER, test x$resover != xno)

AC_CONFIG_FILES([Makefile src/Makefile doc/Makefile])
AC_OUTPUT
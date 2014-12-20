EXTRA_DIST = autogen.sh

lib_LTLIBRARIES = libgshkl.la
libgshkl_la_SOURCES = greshunkel.c
libgshkl_la_LDFLAGS = -version-info 666

bin_PROGRAMS = greshunkel_test
greshunkel_test_SOURCES = greshunkel_test.c
greshunkel_test_LDADD = libgshkl.la

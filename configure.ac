dnl Process this file with autoconf to produce a configure script
AC_INIT
AC_CONFIG_SRCDIR([DESCRIPTION])

dnl Determine R_HOME.
: ${R_HOME=‘R RHOME‘}
if test -z "${R_HOME}"; then
  echo "could not determine R_HOME"
  exit 1
fi

dnl One way is to specify both include and lib paths
AC_ARG_WITH([mpi-include],
            AS_HELP_STRING([--with-mpi-include=INCLUDE_PATH],[location of MPI header files]),
            [mpi_include_path=$withval])
if test -n "${mpi_include_path}" ; then
  MPI_INCLUDE_PATH="${mpi_include_path}"
  if test ! -f "${MPI_INCLUDE_PATH}/mpi.h" ; then
    AC_MSG_ERROR([Value of --with-mpi-include does not contain mpi.h])
  fi
else
  if test -n "${MPI_INCLUDE}" ; then
    MPI_INCLUDE_PATH="${MPI_INCLUDE}"
    if test ! -f "${MPI_INCLUDE_PATH}/mpi.h" ; then
      AC_MSG_ERROR([Value of MPI_INCLUDE does not contain mpi.h])
    fi
    echo "setting mpi include path from MPI_INCLUDE"
  fi
fi

AC_ARG_WITH([mpi-libpath],
            AS_HELP_STRING([--with-mpi-libpath=LIB_PATH],[location of MPI library files]),
            [mpi_lib_path=$withval])
if test -n "${mpi_lib_path}" ; then
  MPI_LIBPATH="${mpi_lib_path}"
else
  if test -n "${MPI_LIB_PATH}" ; then
    MPI_LIBPATH="${MPI_LIB_PATH}"
    echo "setting mpi lib path from MPI_LIB_PATH"
  fi
fi

if test -n "${MPI_INCLUDE_PATH}" && test -z "${MPI_LIBPATH}" ; then
  AC_MSG_ERROR([Must specify both the include and lib paths])
fi
if test -z "${MPI_INCLUDE_PATH}" && test -n "${MPI_LIBPATH}" ; then
  AC_MSG_ERROR([Must specify both the include and lib paths])
fi

MPITYPE="UNKNOWN"

AC_ARG_WITH([mpi-type],
            AS_HELP_STRING([--with-mpi-type=MPI_TYPE],[type of MPI: OPENMPI, MPICH2, LAM, or FPMPI]),
            [mpi_type=$withval])
if test -n "${mpi_type}" ; then
  MPITYPE="${mpi_type}"
else
  if test -n "${MPI_TYPE}" ; then
    MPITYPE="${MPI_TYPE}"
  fi
fi


dnl Check compiler.
CC=`"${R_HOME}/bin/R" CMD config CC`
AC_PROG_CC
if test "X$CC" = "X" ; then
  # Our best guess...
  CC="gcc -std=gnu99"
fi


dnl Check MPI in path.
AC_CHECK_PROG(MPIRUN, mpirun, mpirun, "F")
AC_CHECK_PROG(MPIEXEC, mpiexec, mpiexec, "F")
AC_CHECK_PROG(ORTERUN, orterun, orterun, "F")
AC_CHECK_PROG(SED, sed, sed, "F")
AC_CHECK_PROG(MPICC, mpicc, mpicc, "F")
AC_CHECK_PROG(OMPI_INFO, ompi_info, ompi_info, "F")
AC_CHECK_PROG(MPICH2VERSION, mpich2version, mpich2version, "F")


dnl For OpenMPI
if test -z "${MPI_INCLUDE_PATH}" ; then
  if test "$MPITYPE" = "UNKNOWN" || test "$MPITYPE" = "OPENMPI" ; then
    if test "$SED" != "F" && test "$MPICC" != "F" && test "$OMPI_INFO" != "F" ; then
      echo "found sed, mpicc, and ompi_info ..."
      TMP_INC="F"
      TMP_LIB="F"

      TMP_INC_DIRS=`mpicc --showme:incdirs`
      echo ">> TMP_INC_DIRS = ${TMP_INC_DIRS}"

      TMP_IFS="${IFS}"; IFS=" "
      for dir in ${TMP_INC_DIRS}; do
        echo "checking ${dir} ..."
        if test -f "${dir}/mpi.h" ; then
          echo "found ${dir}/mpi.h ..."
          TMP_INC=${dir}
          break
        fi
      done
      IFS="${TMP_IFS}"

      TMP_LIB_DIRS=`mpicc --showme:libdirs`
      echo ">> TMP_LIB_DIRS = ${TMP_LIB_DIRS}"

      TMP_IFS="${IFS}"; IFS=" "
      for dir in ${TMP_LIB_DIRS}; do
        echo "checking ${dir} ..." 
        if test -f "${dir}/libmpi.so" ; then
          echo "found ${dir}/libmpi.so ..." 
          TMP_LIB=${dir}
          break
        fi
        dnl For Mac OS X
        if test -f "${dir}/libmpi.dylib" ; then
          echo "found ${dir}/libmpi.dylib ..."
          TMP_LIB=${dir}
          break
        fi
      done
      IFS="${TMP_IFS}"

      if test "${TMP_INC}" != "F" && test "${TMP_LIB}" != "F" ; then
        echo "found mpi.h and libmpi.so ..."
        echo ">> TMP_INC = ${TMP_INC}"
        echo ">> TMP_LIB = ${TMP_LIB}"
        MPITYPE="OPENMPI"
        MPI_INCLUDE_PATH="${TMP_INC}"
        MPI_LIBPATH="${TMP_LIB}"
      fi
    fi
  fi
fi


dnl For MPICH2
if test -z "${MPI_INCLUDE_PATH}" ; then
#  if test "$MPITYPE" = "UNKNOWN" || test "$MPITYPE" = "MPICH2" ; then
### MPICH2 users have to specify --with-mpi-type=MPICH2 to get this.
  if test "$MPITYPE" = "MPICH2" ; then
    if test "$SED" != "F" && test "$MPICC" != "F" && test "$MPICH2VERSION" != "F" ; then
      echo "found sed, mpicc, and mpich2version ..."
      TMP_INC="F"
      TMP_LIB="F"

      TMP_CMD=`mpicc -show`
      echo ">> TMP_CMD = ${TMP_CMD}"

      TMP_IFS="${IFS}"; IFS=" "
      for dir in ${TMP_CMD}; do
        dir_inc=`echo ${dir} | sed -e "s/^-I\(.*\)$/\1/"`

        echo "checking ${dir_inc} ..."
        if test -f "${dir_inc}/mpi.h" ; then
          echo "found ${dir_inc}/mpi.h ..."
          TMP_INC=${dir_inc}
          break
        fi
      done
      IFS="${TMP_IFS}"

      TMP_IFS="${IFS}"; IFS=" "
      for dir in ${TMP_CMD}; do
        dir_lib=`echo ${dir} | sed -e "s/^-L\(.*\)$/\1/"`

        echo "checking ${dir_lib} ..." 
        if test -f "${dir_lib}/libmpich.so" ; then
          echo "found ${dir_lib}/libmpich.so ..." 
          TMP_LIB=${dir_lib}
          break
        fi
        dnl For Mac OS X
        if test -f "${dir_lib}/libmpich.dylib" ; then
          echo "found ${dir_lib}/libmpich.dylib ..."
          TMP_LIB=${dir_lib}
          break
        fi
      done
      IFS="${TMP_IFS}"

      if test "${TMP_INC}" != "F" && test "${TMP_LIB}" != "F" ; then
        echo "found mpi.h and libmpich.so ..."
        echo ">> TMP_INC = ${TMP_INC}"
        echo ">> TMP_LIB = ${TMP_LIB}"
        MPITYPE="MPICH2"
        MPI_INCLUDE_PATH="${TMP_INC}"
        MPI_LIBPATH="${TMP_LIB}"
      fi
    fi
  fi
fi


dnl For nothing found from above
dnl 'Paths not specified'
if test -z "${MPI_INCLUDE_PATH}" ; then
  dnl Another way is to specify MPI_ROOT, perhaps via --with-mpi
  AC_ARG_WITH(mpi,
              AS_HELP_STRING([--with-mpi=LIB_PATH],[location of top-level MPI directory]),
              [ if test "${withval}" != "no"; then
                      MPI_ROOT=${withval}
                      MPI_INCLUDE_PATH="${MPI_ROOT}/include"
                      LDFLAGS="$LDFLAGS -L${withval}/lib"
                      CPPFLAGS="$CPPFLAGS -I${withval}/include"
                fi ]
  )

  dnl Detect from conventional paths
  if test -z "$MPI_ROOT" ; then
    for d in /opt/SUNWhpc /opt/SUNWhpc/HPC \
             /opt/SUNWhpc/HPC/sun /opt/SUNWhpc/HPC8.2.1c/sun \
             /opt/SUNWhpc/HPC/gnu /opt/SUNWhpc/HPC8.2.1c/gnu \
             /opt/lib /usr/lib /usr /usr/local/lib /usr/local \
             /opt/lib64 /usr/lib64 /usr/local/lib64 \
             /usr/lib/mpi/gcc/openmpi \
             /usr/lib64/mpi/gcc/openmpi \
             /usr/lib/mpi/gcc/ompi \
             /usr/lib64/mpi/gcc/ompi ; do
      if test -f $d/include/openmpi-i386/mpi.h && test -d $d/lib/openmpi/lib; then
      dnl This is what Fedora 17 had
        echo "I am here $d and it is openmpi"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/openmpi-x86_64/mpi.h && test -d $d/lib/openmpi/lib; then
      dnl This is what Fedora 17 had
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/openmpi-x86_64/mpi.h && test -d $d/lib64/openmpi/lib; then
      dnl This is what Fedora 17 had
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/mpi.h && test -d $d/lib/openmpi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/mpi.h && test -d $d/lib64/openmpi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/openmpi/mpi.h && test -d $d/lib/openmpi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/openmpi/mpi.h && test -d $d/lib64/openmpi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/mpi.h && test -d $d/lib/ompi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/mpi.h && test -d $d/lib64/ompi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/ompi/mpi.h && test -d $d/lib/ompi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/include/ompi/mpi.h && test -d $d/lib64/ompi; then
        echo "I am here $d and it is OpenMPI"
        MPI_ROOT=$d
        MPITYPE="OPENMPI"
        break
      elif test -f $d/openmpi/include/mpi.h && test -d $d/openmpi/lib; then
        echo "I am here $d/openmpi and it is OpenMPI"
        MPI_ROOT=$d/openmpi
        MPITYPE="OPENMPI"
        break
      elif test -f $d/ompi/include/mpi.h && test -d $d/ompi/lib; then
        echo "I am here $d/ompi and it is OpenMPI"
        MPI_ROOT=$d/ompi
        MPITYPE="OPENMPI"
        break
      elif test -f $d/lam/include/mpi.h && test -d $d/lam/lib; then
        echo "I am here $d/lam and it is LAM"
        MPI_ROOT=$d/lam
        MPITYPE="LAM"
        break
      elif test -f $d/include/lam/mpi.h && test -d $d/lib/lam; then
        echo "I am here $d and it is LAM"
        MPI_ROOT=$d
        MPITYPE="LAM"
        break
      elif test -f $d/include/lam/mpi.h && test -d $d/lib64/lam; then
        echo "I am here $d and it is LAM"
        MPI_ROOT=$d
        MPITYPE="LAM"
        break
      elif test -f $d/mpich2/include/mpi.h; then
        echo "I am here $d/mpich2 and it is MPICH2"
        MPI_ROOT=$d/mpich2
        MPITYPE="MPICH2"
        break
      elif test -f $d/mpi/include/mpi.h; then
        echo "I am here $d/mpi"
        MPI_ROOT=$d/mpi
        break
      elif test -f $d/include/mpi.h; then
        echo "I am here $d"
        MPI_ROOT=$d
        break
      fi
    done
  fi


  if test -n "$MPI_ROOT"; then
    dnl Guess MPITYPE if it is unknown and user supplies MPI_ROOT
    if test $MPITYPE = "UNKNOWN" ; then
      if test -d ${MPI_ROOT}/openmpi; then
        MPITYPE="OPENMPI"
      elif test -d ${MPI_ROOT}/ompi; then
        MPITYPE="OPENMPI"
      elif test -d ${MPI_ROOT}/lib/openmpi; then
        MPITYPE="OPENMPI"
      elif test -d ${MPI_ROOT}/lib64/openmpi; then
        MPITYPE="OPENMPI"
      elif test -d ${MPI_ROOT}/lib/ompi; then
        MPITYPE="OPENMPI"
      elif test -d ${MPI_ROOT}/lib64/ompi; then
        MPITYPE="OPENMPI"
      elif test -f ${MPI_ROOT}/lib/liblam.so; then
        MPITYPE="LAM"
      elif test -f ${MPI_ROOT}/lib64/liblam.so; then
        MPITYPE="LAM"
      elif test -d ${MPI_ROOT}/lib/lam; then
        MPITYPE="LAM"
      elif test -d ${MPI_ROOT}/lib64/lam; then
        MPITYPE="LAM"
      elif test -f ${MPI_ROOT}/lib/libmpich.a; then
        MPITYPE="MPICH2"
      elif test -f ${MPI_ROOT}/lib64/libmpich.a; then
        MPITYPE="MPICH2"
      elif test -f ${MPI_ROOT}/lib/libmpich.so; then
        MPITYPE="MPICH2"
      elif test -f ${MPI_ROOT}/lib64/libmpich.so; then
        MPITYPE="MPICH2"
      elif test -f ${MPI_ROOT}/lib/libmpi.so; then
        dnl Nautilus needs this
        MPITYPE="MPT"
      fi
    fi


    echo "Trying to find mpi.h ..."
    if test -f ${MPI_ROOT}/include/openmpi-i386/mpi.h; then
      dnl This is what Fedora 17 had
      echo "Found in ${MPI_ROOT}/include/openmpi-i386"
      MPI_INCLUDE_PATH="${MPI_ROOT}/include/openmpi-i386"
    elif test -f ${MPI_ROOT}/include/openmpi-x86_64/mpi.h; then
      dnl This is what Fedora 17 had
      echo "Found in ${MPI_ROOT}/include/openmpi-x86_64"
      MPI_INCLUDE_PATH="${MPI_ROOT}/include/openmpi-x86_64"
    elif test -f ${MPI_ROOT}/include/openmpi/mpi.h; then
      echo "Found in ${MPI_ROOT}/include/openmpi"
      MPI_INCLUDE_PATH="${MPI_ROOT}/include/openmpi"
    elif test -f ${MPI_ROOT}/include/ompi/mpi.h; then
      echo "Found in ${MPI_ROOT}/include/ompi"
      MPI_INCLUDE_PATH="${MPI_ROOT}/include/ompi"
    elif test -f ${MPI_ROOT}/include/lam/mpi.h; then
      echo "Found in ${MPI_ROOT}/include/lam"
      MPI_INCLUDE_PATH="${MPI_ROOT}/include/lam"
    elif test -f ${MPI_ROOT}/include/mpich2/mpi.h; then
      echo "Found in ${MPI_ROOT}/include/mpich2"
      MPI_INCLUDE_PATH="${MPI_ROOT}/include/mpich2"
    elif test -f ${MPI_ROOT}/include/mpi.h; then
      echo "Found in ${MPI_ROOT}/include"
      MPI_INCLUDE_PATH="${MPI_ROOT}/include"		  
    else 
      echo "Cannot find mpi.h header file"
      echo "Please use --with-mpi=/path/to/mpi or specify the environment variable MPI_ROOT"
      exit 1
    fi
  else
    AC_CHECK_HEADER(mpi.h,, AC_MSG_ERROR(["Cannot find mpi.h header file"]))
  fi 


  echo "Trying to find libmpi.so or libmpich.a ..."
  if test -f ${MPI_ROOT}/libmpi.so; then
    dnl This is what Fedora 10 had
    echo "Found libmpi in ${MPI_ROOT}"
    MPI_LIBPATH="${MPI_ROOT}"
  elif test -f ${MPI_ROOT}/lib/openmpi/lib/libmpi.so; then
    dnl This is what Fedora 17 had
    echo "Found libmpi in ${MPI_ROOT}/lib/openmpi/lib"
    MPI_LIBPATH="${MPI_ROOT}/lib/openmpi/lib"
  elif test -f ${MPI_ROOT}/lib64/openmpi/lib/libmpi.so; then
    dnl This is what Fedora 17 had
    echo "Found libmpi in ${MPI_ROOT}/lib64/openmpi/lib"
    MPI_LIBPATH="${MPI_ROOT}/lib64/openmpi/lib"
  elif test -f ${MPI_ROOT}/lib/openmpi/libmpi.so; then
    echo "Found libmpi in ${MPI_ROOT}/lib/openmpi"
    MPI_LIBPATH="${MPI_ROOT}/lib/openmpi"
  elif test -f ${MPI_ROOT}/lib/openmpi/libmpi.dylib; then
    echo "Found libmpi in ${MPI_ROOT}/lib/openmpi"
    MPI_LIBPATH="${MPI_ROOT}/lib/openmpi"
  elif test -f ${MPI_ROOT}/lib64/openmpi/libmpi.so; then
    echo "Found libmpi in ${MPI_ROOT}/lib64/openmpi"
    MPI_LIBPATH="${MPI_ROOT}/lib64/openmpi"
  elif test -f ${MPI_ROOT}/lib64/openmpi/libmpi.dylib; then
    echo "Found libmpi in ${MPI_ROOT}/lib64/openmpi"
    MPI_LIBPATH="${MPI_ROOT}/lib64/openmpi"
  elif test -f ${MPI_ROOT}/lib/libmpi.so; then
    echo "Found libmpi in ${MPI_ROOT}/lib"
    MPI_LIBPATH="${MPI_ROOT}/lib"
  elif test -f ${MPI_ROOT}/lib/libmpi.dylib; then
    echo "Found libmpi in ${MPI_ROOT}/lib"
    MPI_LIBPATH="${MPI_ROOT}/lib"
  elif test -f ${MPI_ROOT}/lib64/libmpi.so; then
    echo "Found libmpi in ${MPI_ROOT}/lib64"
    MPI_LIBPATH="${MPI_ROOT}/lib64"		
  elif test -f ${MPI_ROOT}/lib64/libmpi.dylib; then
    echo "Found libmpi in ${MPI_ROOT}/lib64"
    MPI_LIBPATH="${MPI_ROOT}/lib64"		
  elif test -f ${MPI_ROOT}/lib/lam/libmpi.so; then
    echo "Found libmpi in ${MPI_ROOT}/lib/lam"
    MPI_LIBPATH="${MPI_ROOT}/lib/lam"
    MPI_INCL2="-I${MPI_INCLUDE_PATH}/32"
  elif test -f ${MPI_ROOT}/lib64/lam/libmpi.so; then
    echo "Found libmpi in ${MPI_ROOT}/lib64/lam"
    MPI_LIBPATH="${MPI_ROOT}/lib64/lam"
    MPI_INCL2="-I${MPI_INCLUDE_PATH}/64"
  elif test -f ${MPI_ROOT}/lib/libmpich.so; then
    echo "Found libmpich in ${MPI_ROOT}/lib"
    MPI_LIBPATH="${MPI_ROOT}/lib"
    AC_CHECK_DECLS([MPICH2, MPICH2],[  MPI_DEFS="-DMPI2" ],, 
                   [#include <mpi.h>])
  elif test -f ${MPI_ROOT}/lib64/libmpich.so; then
    echo "Found libmpich in ${MPI_ROOT}/lib64"
    MPI_LIBPATH="${MPI_ROOT}/lib64"
    AC_CHECK_DECLS([MPICH2, MPICH2],[  MPI_DEFS="-DMPI2" ],, 
                   [#include <mpi.h>])
  elif test -f ${MPI_ROOT}/lib/libmpich.a; then
    echo "Found libmpich in ${MPI_ROOT}/lib"
    MPI_LIBPATH="${MPI_ROOT}/lib"
    AC_CHECK_DECLS([MPICH2, MPICH2],[  MPI_DEFS="-DMPI2" ],, 
                   [#include <mpi.h>])
  elif test -f ${MPI_ROOT}/lib64/libmpich.a; then
    echo "Found libmpich in ${MPI_ROOT}/lib64"
    MPI_LIBPATH="${MPI_ROOT}/lib64"
    AC_CHECK_DECLS([MPICH2, MPICH2],[  MPI_DEFS="-DMPI2" ],, 
                   [#include <mpi.h>])
  else
    AC_CHECK_LIB(mpi, main, MPI_LIBPATH="${MPI_ROOT}/lib",
                 [ echo "libmpi not found. exiting..."
                   exit 1 ])
  fi


  if test "$MPITYPE" = "LAM" ; then
    echo "Try to find liblam.so ..."
    if test -f ${MPI_ROOT}/lib/liblam.so; then
      echo "Found liblam in ${MPI_ROOT}/lib"
      MPI_LIBPATH="${MPI_ROOT}/lib"
    elif test -f ${MPI_ROOT}/lib64/liblam.so; then
      echo "Found liblam in ${MPI_ROOT}/lib64"
      MPI_LIBPATH="${MPI_ROOT}/lib64"
    elif test -f ${MPI_ROOT}/lib/lam/liblam.so; then
      echo "Found liblam in ${MPI_ROOT}/lib/lam"
      MPI_LIBPATH="${MPI_ROOT}/lib/lam"
    elif test -f ${MPI_ROOT}/lib64/lam/liblam.so; then
      echo "Found liblam in ${MPI_ROOT}/lib64/lam"
      MPI_LIBPATH="${MPI_ROOT}/lib64/lam"
    else
      AC_CHECK_LIB(lam, main, [ echo "liblam not found. Probably not LAM-MPI" ])
    fi    
  fi

fi dnl Dnd of 'Paths not specified'




dnl Now we have found the include and lib paths and may know the type

AC_CHECK_LIB(util, openpty, [ MPI_LIBS="$MPI_LIBS -lutil" ])
AC_CHECK_LIB(pthread, main, [ MPI_LIBS="$MPI_LIBS -lpthread" ])

if test "$MPITYPE" = "OPENMPI" || test "$MPITYPE" = "LAM" || test "$MPITYPE" = "MPICH2" || test "$MPITYPE" = "FPMPI"; then
  MPI_DEFS="-DMPI2"
fi

PKG_CPPFLAGS="-I${MPI_INCLUDE_PATH} ${MPI_INCL2} ${MPI_DEFS} -D${MPITYPE}"
case "$MPITYPE" in
  OPENMPI)
    PKG_LIBS="-L${MPI_LIBPATH} -lmpi ${MPI_LIBS}"
    ;;
  LAM)
    PKG_LIBS="-L${MPI_LIBPATH} -lmpi -llam ${MPI_LIBS}"
    ;;
  MPICH)
    PKG_LIBS="-L${MPI_LIBPATH} -lmpich -lmpl -lopa ${MPI_LIBS}"
    ;;
  MPICH2)
    PKG_LIBS="-L${MPI_LIBPATH} -lmpich -lmpl -lopa ${MPI_LIBS}"
    ;;
  FPMPI)
    PKG_LIBS="-L${MPI_LIBPATH} -lfpmpi ${MPI_LIBS}"
    ;;
  *)
    AC_MSG_ERROR([Unknown type of mpi: use --with-mpi-type or MPI_TYPE to specify it])
    ;;
esac


PKG_LIBS="\$(LAPACK_LIBS) \$(BLAS_LIBS) ${PKG_LIBS}"

AC_SUBST(PKG_CPPFLAGS)
AC_SUBST(PKG_LIBS)

AC_CONFIG_FILES([src/Makevars])
AC_OUTPUT

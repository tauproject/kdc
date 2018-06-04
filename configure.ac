# @(#) $Tau: kdc/configure.ac $

## ------------------------------------------------------------------------- ##
##                          T A U    P R O J E C T                           ##
## ------------------------------------------------------------------------- ##
##                                                                           ##
##  Licensed under the Apache License, Version 2.0 (the "License");          ##
##  you may not use this file except in compliance with the License.         ##
##                                                                           ##
##  You may obtain a copy of the License at:                                 ##
##                                                                           ##
##      http://www.apache.org/licenses/LICENSE-2.0                           ##
##                                                                           ##
##  Unless required by applicable law or agreed to in writing, software      ##
##  distributed under the License is distributed on an "AS-IS" BASIS,        ##
##  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. ##
##  See the License for the specific language governing permissions and      ##
##  limitations under the License.                                           ##
##                                                                           ##
## ------------------------------------------------------------------------- ##

AC_INIT([kdc],[shimmer])
AS_VAR_SET([ORG],[tauproject])
AC_SUBST([ORG])
AS_VAR_SET([RELEASE],[shimmer])
AC_SUBST([RELEASE])
AC_CONFIG_AUX_DIR([scripts])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE([foreign])
AM_MAINTAINER_MODE

AC_CHECK_PROG([DOCKER],[docker],[docker])
AM_CONDITIONAL([WITH_DOCKER],[test x"$DOCKER" != x""])

AC_CONFIG_FILES([
Makefile
docker/Makefile
docker/kdc/Makefile
scripts/Makefile
scripts/ci/Makefile
scripts/docker/Makefile
])
	
AC_OUTPUT

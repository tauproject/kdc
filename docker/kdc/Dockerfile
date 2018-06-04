# @(#) $Tau: kdc/docker/kdc/Makefile.am $

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

FROM tauproject/core:shimmer

RUN apt-get install -qq heimdal-servers heimdal-clients heimdal-kdc openssl pwgen

RUN mkdir /app /app/config /app/db /app/etc
COPY docker/kdc/kdc.conf.in /app/etc
COPY docker/kdc/krb5.conf.in /app/etc
COPY docker/kdc/services /etc/services
COPY docker/kdc/inetd.conf /etc/inetd.conf
COPY scripts/docker/entrypoint /app/entrypoint

VOLUME /app/db

EXPOSE 80/tcp
EXPOSE 88/tcp
EXPOSE 88/udp
EXPOSE 464/udp
EXPOSE 749/tcp
EXPOSE 754/tcp

CMD [ "run" ]

ENTRYPOINT [ "/app/entrypoint" ]

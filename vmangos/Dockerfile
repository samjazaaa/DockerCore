FROM ubuntu:22.04 as build

RUN apt-get update && apt-get install -y g++ libace-dev libtbb-dev git cmake libmysqlclient-dev openssl libssl-dev build-essential checkinstall zlib1g-dev
RUN export ACE_ROOT=/usr/include/ace

WORKDIR /vmangos
RUN git clone -b development https://github.com/vmangos/core

# TODO make client build (and extractors?) variable
RUN mkdir build && \
  cd build && \
  cmake /vmangos/core -DDEBUG=0 -DSUPPORTED_CLIENT_BUILD=5875 -DUSE_EXTRACTORS=1 -DCMAKE_INSTALL_PREFIX=/vmangos && make -j4 && make install && \
  make -j4 && \
  make install

FROM ubuntu:22.04 as server-base
RUN apt-get update && apt-get install -y libace-dev libmysqlclient-dev libtbb-dev

WORKDIR /vmangos
RUN mkdir etc

FROM server-base as realmd

COPY --from=build /vmangos/bin/realmd /vmangos/realmd
COPY --from=build /vmangos/etc/realmd.conf.dist /vmangos/etc/realmd.conf

CMD ["./realmd"]

FROM server-base as mangosd

COPY --from=build /vmangos/bin/mangosd /vmangos/mangosd
COPY --from=build /vmangos/etc/mangosd.conf.dist /vmangos/etc/mangosd.conf

CMD ["./mangosd"]

FROM server-base as tools

# TODO: copy extraction tools from build and execute them on volume as command
# COPY --from=build /vmangos/bin/mangosd /vmangos/mangosd
# COPY --from=build /vmangos/etc/mangosd.conf.dist /vmangos/etc/mangosd.conf

# CMD ["./mangosd"]


FROM mysql:lts as db

# TODO read this from .env
ENV MYSQL_ROOT_PASSWORD changeme-root
ENV MYSQL_USER mangos
ENV MYSQL_PASSWORD changeme

COPY prepare-db.sql /docker-entrypoint-initdb.d/

WORKDIR /vmangos
# TODO make this variable?
RUN wget -o world_full.7z https://github.com/brotalnia/database/raw/refs/heads/master/world_full_14_june_2021.7z
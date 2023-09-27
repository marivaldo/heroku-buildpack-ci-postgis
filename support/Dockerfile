FROM heroku/heroku:20
ENV POSTGRESQL_VERSION=14.9

RUN apt-get -q update
RUN apt-get -q -y install curl gcc g++ make wget unzip build-essential libreadline-dev zlib1g-dev flex bison libkrb5-dev libssl-dev libxml2-dev libxslt1-dev uuid-dev

#RUN wget https://ci-database-binary.s3.amazonaws.com/heroku-20/postgresql-${POSTGRESQL_VERSION}.tgz
RUN wget https://ftp.postgresql.org/pub/source/v14.9/postgresql-14.9.tar.gz
RUN tar -xzf postgresql-${POSTGRESQL_VERSION}.tar.gz
RUN mv postgresql-${POSTGRESQL_VERSION} postgresql

ENV POSTGIS_VERSION=3.3.0
ENV GEOS_VERSION=3.9.4
ENV PROJ_VERSION=4.9.2

RUN wget https://postgis.net/stuff/postgis-${POSTGIS_VERSION}.tar.gz
RUN wget https://download.osgeo.org/proj/proj-${PROJ_VERSION}.tar.gz
RUN wget https://download.osgeo.org/geos/geos-${GEOS_VERSION}.tar.bz2

RUN tar xvfz ./postgis-${POSTGIS_VERSION}.tar.gz
RUN tar xvfz ./proj-${PROJ_VERSION}.tar.gz
RUN tar xvfj ./geos-${GEOS_VERSION}.tar.bz2

RUN cd ./postgresql && ./configure --prefix=/postgresql && make && make install
RUN cd ./geos-${GEOS_VERSION} && ./configure --prefix=/postgis-dependencies && make && make install
RUN cd ./proj-${PROJ_VERSION} && ./configure --prefix=/postgis-dependencies && make && make install

RUN export INCLUDE_PATH="/postgis-dependencies/include:$INCLUDE_PATH"
RUN export CPATH="$INCLUDE_PATH"
RUN export CPPPATH="$INCLUDE_PATH"

RUN mkdir -p /app/.indyno/vendor && cp -r postgresql /app/.indyno/vendor/

RUN cd ./postgis-${POSTGIS_VERSION} && ./configure \
  --prefix=/app/.indyno/vendor/postgresql \
  --with-pgconfig=/app/.indyno/vendor/postgresql/bin/pg_config --without-protobuf \
  --without-raster \
  --with-geosconfig=/postgis-dependencies/bin/geos-config \
  --with-projdir=/postgis-dependencies \
  && make

#COPY postgis-extensions-without-raster.Makefile ./postgis-${POSTGIS_VERSION}/extensions/postgis/Makefile
RUN cd ./postgis-${POSTGIS_VERSION}/extensions/postgis && make

RUN tar -zcvf postgis-dependencies.tar.gz ./postgis-dependencies/lib/
RUN tar -zcvf postgis.tar.gz ./postgis-${POSTGIS_VERSION}

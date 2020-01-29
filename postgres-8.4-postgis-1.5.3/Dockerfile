FROM ubuntu:trusty

MAINTAINER yasser AZAIEZ "yaz@atolcd.com"

# ENV
ENV POSTGRES_PASSWORD postgres
ENV DATA_DIR /data
ENV POSTGRES_DATA_FOLDER ${DATA_DIR}/db
ENV PG_VERSION 8.4.0
ENV GEOS_VERSION 3.3.5
ENV PROJ4_VERSION 4.8.0
ENV POSTGIS_VERSION 1.5.3
ENV LOCALE fr_FR
ENV ENCODING UTF-8

WORKDIR /usr/local/

# install dependencies
RUN apt-get update \
  && apt-get install -y build-essential gcc-4.7 libgdal1h libreadline6-dev libxslt-dev libxml2-dev locales zlib1g-dev curl

# locale
RUN localedef -i $LOCALE -c -f $ENCODING -A /usr/share/locale/locale.alias ${LOCALE}.${ENCODING}
RUN locale-gen ${LOCALE}.${ENCODING}

ADD https://ftp.postgresql.org/pub/source/v${PG_VERSION}/postgresql-${PG_VERSION}.tar.bz2 src/
RUN cd src/ ; tar -xjvf postgresql-${PG_VERSION}.tar.bz2
RUN cd src/postgresql-${PG_VERSION} ; ./configure CC='gcc-4.7 -m64' -with-libxml --with-libxslt --prefix=/usr/local --with-pgport=5432
RUN cd src/postgresql-${PG_VERSION} ; make
RUN cd src/postgresql-${PG_VERSION} ; make install
RUN cd src/postgresql-${PG_VERSION}/contrib ; make all
RUN cd src/postgresql-${PG_VERSION}/contrib ; make install
RUN ldconfig

# build & install geos
ADD http://download.osgeo.org/geos/geos-${GEOS_VERSION}.tar.bz2 src/
RUN cd src/ ; tar -xjvf geos-${GEOS_VERSION}.tar.bz2
RUN cd src/geos-${GEOS_VERSION} ; ./configure
RUN cd src/geos-${GEOS_VERSION} ; make
RUN cd src/geos-${GEOS_VERSION} ; make install
RUN ldconfig

# build & install proj4
ADD http://download.osgeo.org/proj/proj-${PROJ4_VERSION}.tar.gz src/
RUN cd src/ ; tar -xvf proj-${PROJ4_VERSION}.tar.gz
RUN cd src/proj-${PROJ4_VERSION} ; ./configure
RUN cd src/proj-${PROJ4_VERSION} ; make
RUN cd src/proj-${PROJ4_VERSION} ; make install
RUN ldconfig

# build & install postgis
ADD http://download.osgeo.org/postgis/source/postgis-${POSTGIS_VERSION}.tar.gz src/
RUN cd src/ ; tar -xvf postgis-${POSTGIS_VERSION}.tar.gz
RUN cd src/postgis-${POSTGIS_VERSION} ; ./configure
RUN cd src/postgis-${POSTGIS_VERSION} ; make
RUN cd src/postgis-${POSTGIS_VERSION} ; make install
RUN ldconfig

# create postgres user
RUN groupadd postgres
RUN useradd -r postgres -g postgres
RUN echo "postgres:${POSTGRES_PASSWORD}" | chpasswd -e

# create common data directory
RUN mkdir -p ${DATA_DIR}

# create postgres data directory
RUN mkdir -p ${POSTGRES_DATA_FOLDER}
RUN chown postgres:postgres ${POSTGRES_DATA_FOLDER}
RUN chmod 700 ${POSTGRES_DATA_FOLDER}

USER postgres
# initdb
RUN initdb \
  --encoding=${ENCODING} \
  --locale=${LOCALE}.${ENCODING} \
  --lc-collate=${LOCALE}.${ENCODING} \
  --lc-monetary=${LOCALE}.${ENCODING} \
  --lc-numeric=${LOCALE}.${ENCODING} \
  --lc-time=${LOCALE}.${ENCODING} \
  -D ${POSTGRES_DATA_FOLDER}

USER root
# conf
RUN echo "host all all 0.0.0.0/0 md5" >> $POSTGRES_DATA_FOLDER/pg_hba.conf
RUN echo "listen_addresses='*'" >> $POSTGRES_DATA_FOLDER/postgresql.conf

USER postgres
RUN pg_ctl -w -D ${POSTGRES_DATA_FOLDER} start \
# set password
  && psql -c "alter role postgres password '${POSTGRES_PASSWORD}'" \
  && createdb template_postgis \
# postgis template
  && psql template_postgis -f src/postgis-${POSTGIS_VERSION}/postgis/postgis.sql \
  && psql template_postgis -f src/postgis-${POSTGIS_VERSION}/spatial_ref_sys.sql \
  && pg_ctl -w -D ${POSTGRES_DATA_FOLDER} stop

USER root
# clean up
RUN rm -rf /var/lib/apt/lists/* src/*

# port
EXPOSE 5432

# data
VOLUME ${DATA_DIR}

CMD su postgres -c "postgres -D ${POSTGRES_DATA_FOLDER}"

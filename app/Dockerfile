ARG DBMS="postgres"
ARG AUTH="localauth"

FROM python:3.9-alpine as builder
ARG DBMS
ARG AUTH

# set work directory
WORKDIR /build

RUN apk update \
 && apk add gcc python3-dev musl-dev jpeg-dev zlib-dev

# install database dependencies
RUN if [[ "${DBMS}" == "postgres" ]]; then apk add --no-cache postgresql-dev; fi
RUN if [[ "${AUTH}" == "openldap" ]]; then apk add --no-cache build-base openldap-dev; fi
RUN if [[ "${AUTH}" == "oauth" ]]; then apk add --no-cache libffi-dev openssl-dev cargo; fi

# install dependencies
COPY rqmts/ rqmts

RUN cp rqmts/requirements.txt requirements.txt; \
    if [[ "${DBMS}" == "postgres" ]]; then cat rqmts/requirements.postgres.txt >> requirements.txt; fi; \
    if [[ "${DBMS}" == "mariadb" ]]; then cat rqmts/requirements.mariadb.txt >> requirements.txt; fi; \
    if [[ "${AUTH}" == "openldap" ]]; then cat rqmts/requirements.openldap.txt >> requirements.txt; fi; \
    if [[ "${AUTH}" == "oauth" ]]; then cat rqmts/requirements.oauth.txt >> requirements.txt; fi;

RUN pip install --prefix=/build/venv -r requirements.txt

FROM python:3.9-alpine
ARG DBMS
ARG AUTH

# app config
ENV ALLOWED_HOSTS='0.0.0.0' DEBUG=0

# create the app user and directories
ENV HOME=/home/app APP_HOME=/home/app/web
RUN mkdir -p ${HOME} \
 && mkdir -p ${APP_HOME} \
 && addgroup -S app \
 && adduser -S app -G app \
 && mkdir -p "${APP_HOME}/app/media"
WORKDIR ${APP_HOME}

# install database dependencies
RUN if [[ "${DBMS}" == "postgres" ]]; then apk add --no-cache libpq; fi
RUN if [[ "${AUTH}" == "openldap" ]]; then apk add --no-cache libldap; fi
RUN if [[ "${AUTH}" == "oauth"]]; then apk add --no-cache libffi-dev openssl-dev; fi

COPY --from=builder /build/venv/ /usr/local/

# copy project
COPY --chown=app:app app ${APP_HOME}
COPY --chown=app:app entrypoint.sh ${APP_HOME}
COPY --chown=app:app manage.py ${APP_HOME}

ENTRYPOINT ["/bin/sh", "entrypoint.sh"]
EXPOSE 8000
USER app

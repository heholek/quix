FROM maven:3.6.3-slim as maven

ARG SBT_VERSION=1.3.7

# Install sbt
RUN \
  curl -L -o sbt-$SBT_VERSION.deb https://dl.bintray.com/sbt/debian/sbt-$SBT_VERSION.deb && \
  dpkg -i sbt-$SBT_VERSION.deb && \
  rm sbt-$SBT_VERSION.deb && \
  apt-get update && \
  apt-get install sbt && \
  sbt sbtVersion

COPY quix-api/src ./build/quix-api/src
COPY quix-core/src ./build/quix-core/src
COPY /quix-modules/quix-presto-module/src ./build/quix-modules/quix-presto-module/src
COPY /quix-modules/quix-athena-module/src ./build/quix-modules/quix-athena-module/src
COPY /quix-modules/quix-bigquery-module/src ./build/quix-modules/quix-bigquery-module/src
COPY /quix-modules/quix-jdbc-module/src ./build/quix-modules/quix-jdbc-module/src
COPY /quix-modules/quix-python-module/src ./build/quix-modules/quix-python-module/src
COPY /quix-modules/quix-dummy-module/src ./build/quix-modules/quix-dummy-module/src
WORKDIR ./build
COPY build.sbt .
COPY version.sbt .
RUN sbt publishM2

COPY /quix-webapps/quix-web-spring/pom.xml ./quix-webapps/quix-web-spring/
COPY /quix-webapps/quix-web-spring/src ./quix-webapps/quix-web-spring/src
WORKDIR ./quix-webapps/quix-web-spring
RUN mvn -T 1C install -am -DskipTests

FROM ubuntu:19.10

# Setup python and java and base system
ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND noninteractive
ENV LANG=en_US.UTF-8

RUN apt-get update && \
  apt-get install -q -y --no-install-recommends \
  default-jdk \
  python3 \
  python-dev \
  python3-dev \
  python3-pip \
  libsnappy-dev \
  language-pack-en \
  build-essential \
  && rm -rf /var/lib/apt/lists/*

RUN pip3 install --no-cache --upgrade pip setuptools wheel py4j

EXPOSE 8081

COPY --from=maven /build/quix-webapps/quix-web-spring/target/quix-web-spring-*.jar ./quix.jar

CMD ["java", "-jar", "quix.jar"]
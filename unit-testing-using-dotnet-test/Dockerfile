FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
#-alpine

COPY . /build
WORKDIR /build

RUN apt-get update && \
    apt-get install -y openjdk-11-jre

WORKDIR /app
COPY --from=build /build/out .

# export ASPNETCORE_URLS=http://localhost:8080
ENV ASPNETCORE_URLS "http://*:8080"

#ENV JAVA_VER=11
#ENV JAVA_HOME="/usr/lib/jvm/java-11-openjdk"

ENV PATH="${PATH}:/root/.dotnet/tools"
ENV PATH="${PATH}:/tmp/usr/lib/jvm/java-11-openjdk"
ENV PATH="${PATH}:/tmp/.dotnet/tools"

## RUN chmod 755 ...sonar-scanner-4.7.0.2747/jre/bin/java
RUN dotnet tool install --global dotnet-sonarscanner
RUN dotnet add package XunitXml.TestLogger --version 3.0.70
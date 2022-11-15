FROM microsoft/dotnet:2.2-sdk AS build

COPY . /build
WORKDIR /build

RUN e

FROM microsoft/dotnet:2.2-aspnetcore-runtime

WORKDIR /app
COPY --from=build /build/out .

# export ASPNETCORE_URLS=http://localhost:8080
ENV ASPNETCORE_URLS "http://*:8080"
EXPOSE 8080

# dotnet ./sample-dotnet-app/sample-dotnet-app.dll
ENTRYPOINT ["dotnet", "sample-dotnet-app.dll"]

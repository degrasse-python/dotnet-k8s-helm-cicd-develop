FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build

COPY . /build
WORKDIR /build

RUN dotnet publish -c Release -o out ./sample-dotnet-app

FROM mcr.microsoft.com/dotnet/aspnet:6.0

WORKDIR /app
COPY --from=build /build/out .

# export ASPNETCORE_URLS=http://localhost:8080
ENV ASPNETCORE_URLS "http://*:8080"
EXPOSE 8080

# dotnet ./sample-dotnet-app/sample-dotnet-app.dll
ENTRYPOINT ["dotnet", "sample-dotnet-app.dll"]

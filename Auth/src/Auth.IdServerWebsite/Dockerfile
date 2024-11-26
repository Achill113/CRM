FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER root

WORKDIR /app
EXPOSE 80
EXPOSE 443

# Copy the certificate into the container and install it
COPY ./Certs/aspnetcore.crt /usr/local/share/ca-certificates/aspnetcore.crt
RUN update-ca-certificates

# Optionally, if you're using a PFX file for HTTPS binding, ensure it is also available
COPY ./Certs/aspnetcore.pfx /https/aspnetcore.pfx

USER $APP_UID

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["Auth/src/Auth.IdServerWebsite/Auth.IdServerWebsite.csproj", "src/Auth.IdServerWebsite/"]
RUN dotnet restore "src/Auth.IdServerWebsite/Auth.IdServerWebsite.csproj"
COPY . .
WORKDIR "/src/Auth/src/Auth.IdServerWebsite"
RUN dotnet build "Auth.IdServerWebsite.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "Auth.IdServerWebsite.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENV ASPNETCORE_URLS=http://*:80;https://*:443
ENTRYPOINT ["dotnet", "Auth.IdServerWebsite.dll"]

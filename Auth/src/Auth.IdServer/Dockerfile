FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
# Switch to root to handle installation
USER root

# Define directories and environment variables
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Copy the certificate into the container and install it
COPY ./Certs/aspnetcore.crt /usr/local/share/ca-certificates/aspnetcore.crt
RUN update-ca-certificates

# Optionally, if you're using a PFX file for HTTPS binding, ensure it is also available
COPY ./Certs/aspnetcore.pfx /https/aspnetcore.pfx

# Change user back if needed
USER $APP_UID

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["Auth/src/Auth.IdServer/Auth.IdServer.csproj", "src/Auth.IdServer/"]
RUN dotnet restore "src/Auth.IdServer/Auth.IdServer.csproj"
COPY . .
WORKDIR "/src/Auth/src/Auth.IdServer"
RUN dotnet build "Auth.IdServer.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "Auth.IdServer.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENV ASPNETCORE_URLS=http://*:80;https://*:443
ENTRYPOINT ["dotnet", "Auth.IdServer.dll"]


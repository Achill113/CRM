FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["src/Auth.IdServer/Auth.IdServer.csproj", "src/Auth.IdServer/"]
RUN dotnet restore "src/Auth.IdServer/Auth.IdServer.csproj"
COPY . .
WORKDIR "/src/src/Auth.IdServer"
RUN dotnet build "Auth.IdServer.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "Auth.IdServer.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENV ASPNETCORE_URLS=http://*:80;https://*:443
ENTRYPOINT ["dotnet", "Auth.IdServer.dll"]

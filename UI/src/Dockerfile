#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 44412

ENV ASPNETCORE_URLS=https://+:44412

# Creates a non-root user with an explicit UID and adds permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-dotnet-configure-containers
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["UI/src/DigiVisio2030.UI.csproj", "UI/src/"]
RUN dotnet restore "UI/src/DigiVisio2030.UI.csproj"
RUN ls -las
COPY . .
RUN ls -las
WORKDIR "/src/UI/src"
RUN ls -las
RUN dotnet build "DigiVisio2030.UI.csproj" -c Release -o /app/build

# Setup Node
ENV NODE_VERSION 16.14.0
ENV NODE_DOWNLOAD_SHA 2c69e7b040c208b61ebf9735c63d2e5bcabfed32ef05a9b8dd5823489ea50d6b
RUN curl -SL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz" --output nodejs.tar.gz \
    && echo "$NODE_DOWNLOAD_SHA nodejs.tar.gz" | sha256sum -c - \
    && tar -xzf "nodejs.tar.gz" -C /usr/local --strip-components=1 \
    && rm nodejs.tar.gz \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs

FROM build AS publish
RUN dotnet publish "DigiVisio2030.UI.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
COPY UI/src/DigiVisio2030.UI.pem .
COPY UI/src/DigiVisio2030.UI.key .
ENTRYPOINT ["dotnet", "DigiVisio2030.UI.dll"]
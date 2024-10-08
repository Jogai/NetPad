# Stage N: Build .NET Application
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS dotnet-builder

# Set the working directory
WORKDIR /app

# Copy the .NET project files
COPY ./ ./

RUN dotnet restore
RUN dotnet build ./ --no-restore --configuration Release -v q
# Copy the rest of the .NET app files
COPY ./ ./

# Build the .NET application
RUN dotnet publish ./Apps/NetPad.Apps.App/NetPad.Apps.App.csproj -c Release -o publish

# Stage A: Build Angular Application
FROM node:lts AS angular-builder

# Set the working directory
WORKDIR /app

# Copy the Angular app files
COPY Apps/NetPad.Apps.App/App ./

# Install Angular dependencies
RUN npm i --prefer-dedupe

# Prevent cache
ADD "https://www.random.org/cgi-bin/randbyte?nbytes=10&format=h" skipcache

# Build the Angular app
RUN npm run build-web

# Stage F: Final Image
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS final
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
	ASPNETCORE_ENVIRONMENT=Production \
	DOTNET_CLI_TELEMETRY_OPTOUT=1 \
	ASPNETCORE_HTTP_PORTS="5001" \
	TZ=UTC
RUN apk add --no-cache icu-libs

# Set the working directory
WORKDIR /app

# Copy Angular build artifacts to serve using a static file server
COPY --from=angular-builder /app/dist /app/wwwroot

# Copy the .NET app build artifacts
COPY --from=dotnet-builder /app/publish .

# Expose the necessary ports
EXPOSE 80 443 5001

# Set the entry point for the .NET application
ENTRYPOINT ["dotnet", "NetPad.Apps.App.dll"]

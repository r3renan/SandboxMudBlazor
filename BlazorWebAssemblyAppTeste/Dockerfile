FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY BlazorWebAssemblyAppTeste.csproj .
RUN dotnet restore BlazorWebAssemblyAppTeste.csproj
COPY . .
RUN dotnet build BlazorWebAssemblyAppTeste.csproj -c Release -o /app/build

FROM build AS publish
RUN dotnet publish BlazorWebAssemblyAppTeste.csproj -c Release -o /app/publish

FROM nginx:alpine AS final
WORKDIR /usr/share/nginx/html
COPY --from=publish /app/publish/wwwroot .
COPY nginx.conf /etc/nginx/conf.d/default.conf
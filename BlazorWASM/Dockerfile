FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY BlazorWASM.csproj .
RUN dotnet restore BlazorWASM.csproj
COPY . .
RUN dotnet build BlazorWASM.csproj -c Release -o /app/build

FROM build AS publish
RUN dotnet publish BlazorWASM.csproj -c Release -o /app/publish

FROM nginx:alpine AS nginxSetup
WORKDIR /usr/share/nginx/html
COPY --from=publish /app/publish/wwwroot .
COPY nginx.conf /etc/nginx/conf.d/default.conf
CMD sed -i -e 's/$PORT/'"$PORT"'/g' /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'
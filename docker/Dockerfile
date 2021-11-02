FROM mcr.microsoft.com/dotnet/sdk:5.0-focal-arm64v8 as build-env

WORKDIR /root/hello
RUN dotnet new console && \
    dotnet build && \
    dotnet publish -r linux-arm64 -p:PublishSingleFile=true -p:InvariantGlobalization=true

FROM arm64v8/ubuntu
COPY --from=build-env /root/hello/bin/Debug/net5.0/linux-arm64/publish/hello /usr/local/bin/
COPY loop.sh /
RUN apt-get update && \
    apt-get install -y libgssapi-krb5-2
CMD ["sh", "/loop.sh"]

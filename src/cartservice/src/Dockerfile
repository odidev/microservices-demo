# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# https://mcr.microsoft.com/v2/dotnet/sdk/tags/list
FROM mcr.microsoft.com/dotnet/sdk:7.0.203 as builder
WORKDIR /app
COPY cartservice.csproj .
RUN if [ `uname -m` =="aarch64" ] ; then \
            dotnet restore cartservice.csproj -r linux-musl-arm64; \
    else \
            dotnet restore cartservice.csproj -r linux-musl-x64; \
    fi
COPY . .
RUN if [ `uname -m` =="aarch64" ] ; then \
            dotnet publish cartservice.csproj -p:PublishSingleFile=true -r linux-musl-arm64 --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c release -o /cartservice --no-restore;\
    else \
            dotnet publish cartservice.csproj -p:PublishSingleFile=true -r linux-musl-x64 --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c release -o /cartservice --no-restore;\
    fi

# https://mcr.microsoft.com/v2/dotnet/runtime-deps/tags/list
FROM mcr.microsoft.com/dotnet/runtime-deps:7.0.4 as without-grpc-health-probe-bin

WORKDIR /app
COPY --from=builder /cartservice .
EXPOSE 7070
ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_URLS=http://*:7070
USER 1000
ENTRYPOINT ["/app/cartservice"]

FROM without-grpc-health-probe-bin
USER root

# renovate: datasource=github-releases depName=grpc-ecosystem/grpc-health-probe
RUN apt-get update && apt-get install wget -y
# renovate: datasource=github-releases depName=grpc-ecosystem/grpc-health-probe
RUN if [ `uname -m` = "aarch64" ] ; then \
            wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.14/grpc_health_probe-linux-arm64; \
        chmod +x /bin/grpc_health_probe; \
    else \
            wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.14/grpc_health_probe-linux-amd64; \
        chmod +x /bin/grpc_health_probe; \
    fi
USER 1000


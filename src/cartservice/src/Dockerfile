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
ARG IMG

# https://mcr.microsoft.com/v2/dotnet/sdk/tags/list
FROM mcr.microsoft.com/dotnet/sdk:$IMG as builder
WORKDIR /app
COPY cartservice.csproj .
ARG arch
RUN if [ `uname -m` = "aarch64" ] ; then \
            arch=arm64; \
    else \
            arch=x64; \
    fi
RUN dotnet restore cartservice.csproj -r linux-musl-$arch
COPY . .
RUN dotnet publish cartservice.csproj -p:PublishSingleFile=true -r linux-musl-$arch --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c release -o /cartservice --no-restore

# https://mcr.microsoft.com/v2/dotnet/runtime-deps/tags/list
ARG tag
RUN if [ `uname -m` = "aarch64" ] ; then \
            tag=7.0.3-alpine3.16-arm64v8; \
    else \
            tag=7.0.4-alpine3.16-amd64@sha256:7141eea9c7be5f4d2f09df427ba37620e50be150fc93015288b3e26c5071af81; \
    fi
FROM mcr.microsoft.com/dotnet/runtime-deps:$tag  as without-grpc-health-probe-bin

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
RUN if [ `uname -m` = "aarch64" ] ; then \
            wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.14/grpc_health_probe-linux-arm64; \
        chmod +x /bin/grpc_health_probe; \
    else \
            wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.14/grpc_health_probe-linux-amd64; \
        chmod +x /bin/grpc_health_probe; \
    fi
USER 1000

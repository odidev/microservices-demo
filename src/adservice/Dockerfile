# Copyright 2020 Google LLC
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

FROM eclipse-temurin:19 as builder

WORKDIR /app

COPY ["build.gradle", "gradlew", "./"]
COPY gradle gradle
RUN chmod +x gradlew
RUN ./gradlew downloadRepos

COPY . .
RUN chmod +x gradlew
RUN ./gradlew installDist

FROM eclipse-temurin:19.0.2_7-jre-focal as without-grpc-health-probe-bin

RUN apk add --no-cache ca-certificates

# Download Stackdriver Profiler Java agent
RUN mkdir -p /opt/cprof && \
    ARCH="$(dpkg --print-architecture)"; \
    if ["${ARCH}"==aarch64|arm64]; then \
        git clone https://github.com/GoogleCloudPlatform/cloud-profiler-java.git; \
        cd cloud-profiler-java; \
        ./build.sh -m arm64; \
        tar xzv -C /opt/cprof && \
        rm -rf cloud-profiler-java; \
        rm -rf profiler_java_agent.tar.gz; \
    else \
        wget -q -O- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent_alpine.tar.gz; \
        tar xzv -C /opt/cprof; \
        rm -rf profiler_java_agent.tar.gz; \
        fi
WORKDIR /app
COPY --from=builder /app .

EXPOSE 9555
ENTRYPOINT ["/app/build/install/hipstershop/bin/AdService"]

FROM without-grpc-health-probe-bin

# renovate: datasource=github-releases depName=grpc-ecosystem/grpc-health-probe
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.14 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-arm64 && \
    chmod +x /bin/grpc_health_probe

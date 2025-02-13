# Copyright 2022 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM golang:1.20-alpine as builder

ARG GOPROXY
ARG GOTAGS
ARG GOGCFLAGS

ARG DRAGONFLY2_GITHUB_BRANCH=release-2.1.9
ARG MANAGER_CONSOLE_BRANCH=release-0.1.7

WORKDIR /go/src/d7y.io/dragonfly/v2

RUN apk --no-cache add bash make gcc libc-dev git && \
    git clone -b ${DRAGONFLY2_GITHUB_BRANCH} --recurse-submodules https://github.com/dragonflyoss/Dragonfly2.git /go/src/d7y.io/dragonfly/v2

RUN make build-scheduler && make install-scheduler

FROM marketplace.gcr.io/google/c2d-debian11 as health

ARG GRPC_HEALTH_PROBE_VERSION=v0.4.11

RUN apt update && apt install -y wget && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

FROM marketplace.gcr.io/google/c2d-debian11

ENV C2D_RELEASE 2.1.9

ENV DRAGONFLY_REDIS_PW dragonfly
ENV DRAGONFLY_REDIS_HOST localhost
ENV DRAGONFLY_MANAGER_ADDR localhost


ENV PATH=/opt/dragonfly/bin:$PATH
RUN echo "hosts: files dns" > /etc/nsswitch.conf

COPY --from=builder /opt/dragonfly/bin/scheduler /opt/dragonfly/bin/scheduler
COPY --from=health /bin/grpc_health_probe /bin/grpc_health_probe

COPY config/scheduler.yaml /etc/dragonfly/scheduler.yaml
COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN apt update && apt install -y gettext-base && \
    chmod +x /docker-entrypoint.sh

EXPOSE 8002

ENTRYPOINT ["/docker-entrypoint.sh"]

FROM microsoft/dotnet:2.1-sdk-alpine as builder
WORKDIR /app
COPY . .
RUN dotnet restore && \
    dotnet build && \
    dotnet publish -c release -r linux-musl-x64 -o /cartservice

# New Relic APM
# Docs: https://docs.newrelic.com/docs/agents/net-agent/installation/install-net-agent-linux
# Releases: https://download.newrelic.com/dot_net_agent/latest_release
RUN wget https://download.newrelic.com/dot_net_agent/latest_release/newrelic-netcore20-agent_8.24.244.0_amd64.tar.gz && \
    tar zxvf newrelic-netcore20-agent_8.24.244.0_amd64.tar.gz && \
    mv newrelic-netcore20-agent /usr/local/
ENV CORECLR_ENABLE_PROFILING=1 \
    CORECLR_PROFILER={36032161-FFC0-4B61-B559-F6C5D41BAE5A} \
    CORECLR_NEWRELIC_HOME=/usr/local/newrelic-netcore20-agent \
    CORECLR_PROFILER_PATH=/usr/local/newrelic-netcore20-agent/libNewRelicProfiler.so \
    NEW_RELIC_APP_NAME=CartService \
    NEW_RELIC_DISTRIBUTED_TRACING_ENABLED=true

# cartservice
FROM alpine:3.8

RUN GRPC_HEALTH_PROBE_VERSION=v0.2.0 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

# Dependencies for runtime
# busybox-extras => telnet
RUN apk add --no-cache \
    busybox-extras \
    libc6-compat \
    libunwind \
    libuuid \
    libgcc \
    libstdc++ \
    libintl \
    icu
WORKDIR /app
COPY --from=builder /cartservice .
ENTRYPOINT ["./cartservice", "start"]

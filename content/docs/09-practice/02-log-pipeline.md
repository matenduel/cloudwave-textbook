---
title: "Practice-2"
weight: 92
draft: false
---


### [실습 2-1] `log` 시스템 구축하기

> Loki + Promtail은 ELK, EFK와 마찬가지로 애플리케이션 로그를 수집하고 조회하기 위한 로그 시스템 구성 방식 중 하나입니다.

- 다음 조건을 만족하는 `Loki` 컨테이너를 실행하세요.

    - 컨테이너 이름은 `loki`로 설정하세요.

    - `grafana/loki:3.4.1` 이미지를 사용하세요.

    - `bridge`드라이버를 사용하는 `observe` 네트워크를 사용하세요.

    - 다음 `loki-config.yaml` 파일을 `bind mount` 방식을 통해 컨테이너의 `/mnt/config` 폴더에 위치시키세요.

      ```yaml
      # loki-config.yaml
      auth_enabled: false
      
      server:
        http_listen_port: 3100
        grpc_listen_port: 9096
        log_level: debug
        grpc_server_max_concurrent_streams: 1000
      
      common:
        instance_addr: 127.0.0.1
        path_prefix: /tmp/loki
        storage:
          filesystem:
            chunks_directory: /tmp/loki/chunks
            rules_directory: /tmp/loki/rules
        replication_factor: 1
        ring:
          kvstore:
            store: inmemory
      
      query_range:
        results_cache:
          cache:
            embedded_cache:
              enabled: true
              max_size_mb: 100
      
      limits_config:
        metric_aggregation_enabled: true
      
      schema_config:
        configs:
          - from: 2020-10-24
            store: tsdb
            object_store: filesystem
            schema: v13
            index:
              prefix: index_
              period: 24h
      
      pattern_ingester:
        enabled: true
        metric_aggregation:
          loki_address: localhost:3100
      
      ruler:
        alertmanager_url: http://localhost:9093
      
      frontend:
        encoding: protobuf
       
      analytics:
        reporting_enabled: false
      ```

    - 다음 커맨드를 컨테이너 실행시 인자로 주입하세요

      ```shell
      --config.file=/mnt/config/loki-config.yaml
      ```

- 다음 조건을 만족하는 `Promtail` 컨테이너를 실행하세요.

    - 컨테이너 이름은 `promtail`로 설정하세요.

    - `grafana/promtail:3.4.1` 이미지를 사용하세요.

    - `bridge`드라이버를 사용하는 `observe` 네트워크를 사용하세요.

    - 다음 `promtail-docker.yaml` 파일을 `bind mount` 방식을 통해 컨테이너의 `/mnt/config` 폴더에 위치시키세요.

      ```yaml
      # promtail-docker.yaml
      server:
        http_listen_port: 9080
        grpc_listen_port: 0
      positions:
        filename: /tmp/positions.yaml
      clients:
        - url: http://loki:3100/loki/api/v1/push
      scrape_configs:
        - job_name: docker
          docker_sd_configs:
            - host: unix:///var/run/docker.sock
              refresh_interval: 5s
          relabel_configs:
            - source_labels: ["__meta_docker_container_name"]
              regex: "/(.*)"
              target_label: "container"
            - source_labels: ["__meta_docker_container_log_stream"]
              target_label: "stream"
      ```



- `/var/run/docker.sock`을 `bind mount` 방식을 통해 컨테이너의 `/var/run/docker.sock`에 바인드 하세요.

- 다음 커맨드를 컨테이너 실행시 인자로 주입하세요.

  ```sh
  --config.file=/mnt/config/promtail-docker.yaml
  ```

- 다음 조건을 만족하는 `grafana` 컨테이너를 실행하세요.

    - 컨테이너 이름은 `grafana`로 설정하세요.

    - `grafana/grafana:12.3.1` 이미지를 사용하세요.

    - `bridge`드라이버를 사용하는 `observe` 네트워크를 사용하세요.

    - `host`에서 3000 포트로 접근할 수 있도록 포트를 `publish` 하세요.

    - 다음과 같이 환경 변수를 설정해주세요.

      ```
      GF_SECURITY_ADMIN_USER=admin
      GF_SECURITY_ADMIN_PASSWORD=admin
      ```

- `grafana`에서 로그를 확인해주세요.

    - `Drilldown > Logs`
    - http://localhost:3000/a/grafana-lokiexplore-app/explore



### [Extra] Loki 로그 시스템 구성 개선하기

- 다음 조건을 만족하도록 `loki` 컨테이너를 수정하세요
    - `Docker volume` 또는 `bind mount`를 이용하여 `/tmp/loki`에 저장된 로그 데이터가 유실되지 않도록 설정하세요.
- 다음 조건을 만족하도록 `promtail` 컨테이너를 수정하세요
    - 중복 로그 수집 방지를 위해 `Docker volume` 또는 `bind mount`를 이용하여 `/tmp`에 저장된 데이터가 유실되지 않도록 설정하세요.



# Monitoring a Java App - heartbeat

## set the credentials

```bash
export CLOUD_AUTH="elastic:toto"
export CLOUD_ID="toto"
```

## create an API key and update heartbeat.yml

```json
POST /_security/api_key
{
  "name": "heartbeat_tutorial_monitor_app_java",
  "role_descriptors": {
    "heartbeat_writer": {
      "cluster": ["monitor", "read_ilm"],
      "index": [
        {
          "names": ["heartbeat-*"],
          "privileges": ["view_index_metadata", "create_doc"]
        }
      ]
    }
  }
}
```

## execute the setup

```bash
docker run \
docker.elastic.co/beats/heartbeat:7.10.2 \
setup \
  -E cloud.id=$CLOUD_ID \
  -E cloud.auth="$CLOUD_AUTH"
```

## run metricbeat

```bash
docker run -d --rm\
  --name=heartbeat \
  --user=root \
  --volume="$(pwd)/heartbeat.yml:/usr/share/heartbeat/heartbeat.yml:ro" \
  docker.elastic.co/beats/heartbeat:7.10.2 heartbeat -e -strict.perms=false \
  -E cloud.id="$CLOUD_ID"
  -E cloud.auth="$CLOUD_AUTH"
```

## generate workload

```bash
docker run skandyla/wrk -c 100 -t 20 -d 5m http://host.docker.internal:7000/wait
```

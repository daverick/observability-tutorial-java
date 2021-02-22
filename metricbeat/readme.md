# Monitoring a Java App - Metricbeat

## set the credentials

```bash
export CLOUD_AUTH="elastic:toto"
export CLOUD_ID="toto"
```

## create an API key and update metricbeat.yml

```json
POST /_security/api_key
{
  "name": "metricbeat_tutorial_monitor_app_java", 
  "role_descriptors": {
    "metricbeat_writer": { 
      "cluster": ["monitor", "read_ilm"],
      "index": [
        {
          "names": ["metricbeat-*"],
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
docker.elastic.co/beats/metricbeat:7.10.2 \
setup \
-E cloud.id="$CLOUD_ID" \
-E cloud.auth="$CLOUD_AUTH"
```

## run metricbeat

```bash
docker run -d --rm\
  --name=metricbeat \
  --user=root \
  --volume="$(pwd)/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:ro" \
  docker.elastic.co/beats/metricbeat:7.10.2 metricbeat -e -strict.perms=false \
  -E cloud.id="$CLOUD_ID"
```

## generate workload

```bash
docker run skandyla/wrk -c 100 -t 20 -d 5m http://host.docker.internal:7000/wait
```

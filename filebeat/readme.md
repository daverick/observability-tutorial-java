# Monitoring a Java App - Metricbeat

## set the credentials

```bash
export CLOUD_AUTH="elastic:toto"
export CLOUD_ID="toto"
```

## create an API key and update filebeat.yml

```json
POST /_security/api_key
{
  "name": "filebeat_tutorial_monitor_app_java", 
  "role_descriptors": {
    "filebeat_writer": { 
      "cluster": ["monitor", "read_ilm"],
      "index": [
        {
          "names": ["filebeat-*"],
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
docker.elastic.co/beats/filebeat:7.10.2 \
setup \
  -E cloud.id="$CLOUD_ID" \
  -E cloud.auth=$CLOUD_AUTH
```

## run filebeat

```bash
docker run -d --rm\
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="$(pwd)/../app/logs/javalin:/tmp/javalin:ro" \
  docker.elastic.co/beats/filebeat:7.10.2 filebeat -e -strict.perms=false \
  -E cloud.id="$CLOUD_ID"
```

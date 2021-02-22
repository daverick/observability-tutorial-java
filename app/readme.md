# Observability tutorial: Monitor a Java Application

Following instructions:
[https://www.elastic.co/guide/en/observability/7.11/monitor-java-app.html]

To enter a graddle env:

```bash
docker run -it --rm -u gradle -v "$PWD":/home/gradle/project -w /home/gradle/project gradle:jdk15 /bin/bash
```

then in the container execute:

```bash
./gradlew clean check
./gradlew assemble shadowJar
```

To execute the app:

```bash
docker run --rm --name my-app \
            -v "$(PWD)"/build:/build \
            -v "$(PWD)"/logs:/tmp/ \
            -p 7000:7000 \
            openjdk:15 java -jar /build/libs/javalin-app-all.jar
```

With the APM agent onto the code:

```bash
export APM_TOKEN="TOTO"
export APM_ENDPOINT_URL="TOTO"
```

```bash
docker run --rm --name my-app \
        -v "$(PWD)"/build:/build \
        -p 7000:7000 \
        -v "$(PWD)"/logs:/tmp/ \
        openjdk:15 java -javaagent:/build/elastic-apm-agent-1.17.0.jar \
                        -Delastic.apm.service_name=javalin-app \
                        -Delastic.apm.application_packages=de.spinscale.javalin \
                        -Delastic.apm.server_urls=$APM_ENDPOINT_URL\
                        -Delastic.apm.secret_token=$APM_TOKEN \
                        -jar build/libs/javalin-app-all.jar
```

with programmatic setup:

```bash
docker run --rm --name my-app \
        -v "$(PWD)"/build:/build \
        -p 7000:7000 \
        -v "$(PWD)"/logs:/tmp/ \
        openjdk:15 java -Delastic.apm.service_name=javalin-app \
                        -Delastic.apm.server_urls=$APM_ENDPOINT_URL\
                        -Delastic.apm.secret_token=$APM_TOKEN \
                        -jar build/libs/javalin-app-all.jar
```

trace method

```bash
docker run --rm --name my-app \
        -v "$(PWD)"/build:/build \
        -p 7000:7000 \
        -v "$(PWD)"/logs:/tmp/ \
        openjdk:15 java -Delastic.apm.service_name=javalin-app \
                        -Delastic.apm.server_urls=$APM_ENDPOINT_URL\
                        -Delastic.apm.secret_token=$APM_TOKEN \
                        -Delastic.apm.trace_methods="de.spinscale.javalin.Log4j2RequestLogger#handle" \
                        -jar build/libs/javalin-app-all.jar
```

with profiling inferred
ELASTIC_APM_PROFILING_INFERRED_SPANS_ENABLED=true

```bash
docker run --rm --name my-app \
        -v "$(PWD)"/build:/build \
        -p 7000:7000 \
        -v "$(PWD)"/logs:/tmp/ \
        openjdk:15 java -Delastic.apm.service_name=javalin-app \
                        -Delastic.apm.server_urls=$APM_ENDPOINT_URL\
                        -Delastic.apm.secret_token=$APM_TOKEN \
                        -Delastic.apm.profiling_inferred_spans_enabled=true \
                        -jar build/libs/javalin-app-all.jar
```

Log correlation

ELASTIC_APM_ENABLE_LOG_CORRELATION=true
```bash
docker run --rm --name my-app \
        -v "$(PWD)"/build:/build \
        -p 7000:7000 \
        -v "$(PWD)"/logs:/tmp/ \
        openjdk:15 java -Delastic.apm.service_name=javalin-app \
                        -Delastic.apm.server_urls=$APM_ENDPOINT_URL\
                        -Delastic.apm.secret_token=$APM_TOKEN \
                        -Delastic.apm.profileing_inferred_spans_enabled=true \
                        -Delastic.apm.enable_log_correlation=true \
                        -jar build/libs/javalin-app-all.jar
```
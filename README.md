# Cloud Native Development with Quarkus (with Panache)

This repository contains some of the code from the _Cloud Native Development with Quarkus - Experienced_ lab exercises.

Use this repo to learn Quarkus and some Java. 

## Build on local machine 

```
git clone https://github.com/sjbylo/quarkus-learning 

cd quarkus-learning

mvn clean package
```

## Build & run in dev mode without tests

```
mvn clean quarkus:dev -DskipTests
```

## Run just the tests

```
mvn clean test
```

## Run and test the app 

```
mvn clean quarkus:dev

curl http://localhost:8080/person | jq

curl http://localhost:8080/person/eyes/BLUE | jq 

curl http://localhost:8080/person/birth/before/1990 | jq

curl "http://localhost:8080/person/datatable?draw=1&start=0&length=10&search\[value\]=yan" | jq

curl "http://localhost:8080/person/datatable?draw=1&start=0&length=2&search\[value\]=F" | jq
```

## List all the Quarkus extensions 

```
mvn quarkus:list-extensions
```

## Add extensions 

```
mvn quarkus:add-extension -Dextensions="smallrye-metrics"

mvn quarkus:add-extension -Dextensions="smallrye-health"

mvn quarkus:add-extension -Dextensions="hibernate-orm-panache,jdbc-h2,jdbc-postgresql,resteasy-jsonb"
```

## Build & deploy uber jar 

```
mvn clean package -DskipTests -Dquarkus.package.type=uber-jar
```

## Build jar 

```
mvn clean package
ls -1 target/*.jar
```

## Run jar locally on different port 

```
java -Dquarkus.http.port=8081 -jar target/*-runner.jar
```

---------

# Build & run on OpenShift 

## Log into OpenShift 

```
OPENSHIFT_API_URL=https://api.openshift.example.com:6443/
oc login $OPENSHIFT_API_URL --insecure-skip-tls-verify=true
oc whoami
```

## "Binary" Build on OpenShift 

```
# These commands will upload the runner jar into a openjdk-11 container and commit it to the built-in registry
oc new-build registry.access.redhat.com/openjdk/openjdk-11-rhel7:1.1 --binary --name=people -l app=people
oc start-build people --from-file target/*-runner.jar --follow
```

## Test running app

```
PEOPLE_ROUTE_URL=$(oc get route people -o=template --template='{{.spec.host}}')

curl http://${PEOPLE_ROUTE_URL}/hello
```

## Deploy with dababase 

```
oc new-app     -e POSTGRESQL_USER=sa     -e POSTGRESQL_PASSWORD=sa     -e POSTGRESQL_DATABASE=person     --name=postgres-database     openshift/postgresql
oc rollout status -w deployment/people
```

## Deploy to OpenShift

```
oc new-app people && oc expose svc/people
```

## Wait for deployment rollout 

```
oc rollout status -w deployment/people
```

## Test the app

```
PEOPLE_ROUTE_URL=$(oc get route people -o=template --template='{{.spec.host}}')
curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2000

curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2000; echo 

curl "http://${PEOPLE_ROUTE_URL}/person/datatable?draw=1&start=0&length=2&search\[value\]=F" | jq

curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2000 | jq
curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2020 | jq
```

## Configure Kube probes 

```
oc set probe deployment/people --readiness --initial-delay-seconds=30 --get-url=http://:8080/health/ready
oc set probe deployment/people --liveness --initial-delay-seconds=30 --get-url=http://:8080/health/live
oc rollout status -w deployment/people
```

## Test this URL in browser 

```
echo "http://${PEOPLE_ROUTE_URL}/datatable.html" ; echo
```


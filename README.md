# Cloud Native Development with Quarkus

This repository contains some of the code from the _Cloud Native Development with Quarkus - Experienced_ lab exercises.

# Some useful commands 

```
git clone https://github.com/sjbylo/quarkus-learning 

cd quarkus-learning

mvn clean package

mvn quarkus:add-extension -Dextensions="smallrye-health"
mvn clean quarkus:dev -DskipTests

mvn clean test

mvn quarkus:generate-config

mvn clean quarkus:dev

mvn clean package -Dquarkus.package.type=uber-jar
```

# Log into OpenShift 

```
OPENSHIFT_API_URL=https://api.openshift.example.com:6443/
oc login $OPENSHIFT_API_URL --insecure-skip-tls-verify=true
oc whoami
```

# Deploy to OpenShift

```
oc new-app people && oc expose svc/people
```

# Wait for deployment rollout 

```
oc rollout status -w deployment/people
```

# Test running app

```
PEOPLE_ROUTE_URL=$(oc get route people -o=template --template='{{.spec.host}}')

curl http://${PEOPLE_ROUTE_URL}/hello
```

# Configure Kube probes 

```
oc set probe deployment/people --readiness --initial-delay-seconds=30 --get-url=http://:8080/health/ready
oc set probe deployment/people --liveness --initial-delay-seconds=30 --get-url=http://:8080/health/live
oc rollout status -w deployment/people
```

# List all the Quarkus extensions 

```
mvn quarkus:list-extensions
```

# Add extensions 

```
mvn quarkus:add-extension -Dextensions="smallrye-metrics"

mvn quarkus:add-extension -Dextensions="hibernate-orm-panache,jdbc-h2,jdbc-postgresql,resteasy-jsonb"

mvn compile quarkus:dev
```

# Build & deploy uber jar 

```
mvn clean package -DskipTests -Dquarkus.package.type=uber-jar
```

# Build
```
oc start-build people --from-file target/*-runner.jar --follow
```


---------
# Build jar 

```
mvn clean package
ls -1 target/*.jar
```

# Run jar locally on different port 

```
java -Dquarkus.http.port=8081 -jar target/*-runner.jar

LAB_PROJECT_NAME=$(oc projects| grep "quarkus-experienced-lab" | head -n 1 | awk '{print $2}')
echo $LAB_PROJECT_NAME 
```

# "Binary" Build on OpenShift 

```
oc new-build registry.access.redhat.com/openjdk/openjdk-11-rhel7:1.1 --binary --name=people -l app=people
oc start-build people --from-file target/*-runner.jar --follow

curl http://localhost:8080/person|jq

curl http://localhost:8080/person/eyes/BLUE

curl http://localhost:8080/person/birth/before/1990

curl "http://localhost:8080/person/datatable?draw=1&start=0&length=10&search\[value\]=yan" | jq

curl "http://localhost:8080/person/datatable?draw=1&start=0&length=2&search\[value\]=F" | jq
```

# Deploy with dababase 

```
oc new-app     -e POSTGRESQL_USER=sa     -e POSTGRESQL_PASSWORD=sa     -e POSTGRESQL_DATABASE=person     --name=postgres-database     openshift/postgresql
oc rollout status -w deployment/people
```

# Test the app

```
PEOPLE_ROUTE_URL=$(oc get route people -o=template --template='{{.spec.host}}')
curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2000

oc get route people -o=template --template='{{.spec.host}}'; echo 

curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2000; echo 

curl "http://localhost:8080/person/datatable?draw=1&start=0&length=2&search\[value\]=F" | jq

curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2000 | jq
curl http://${PEOPLE_ROUTE_URL}/person/birth/before/2020 | jq
```

# Test in browser 

```
echo "http://${PEOPLE_ROUTE_URL}/datatable.html" ; echo
```




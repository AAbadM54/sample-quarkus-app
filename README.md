# Serverless

[Why and When you need to consider OpenShift Serverless](https://www.openshift.com/blog/why-and-when-you-need-to-consider-openshift-serverless)

[Why choose Red Hat OpenShift Serverless?](https://www.redhat.com/en/topics/microservices/why-choose-openshift-serverless)

# OpenShift Serverless

[Install OpenShift Serverless Operator](https://www.redhat.com/en/blog/hands-introduction-openshift-serverless)

[Serverless applications made faster and simpler with OpenShift Serverless GA](https://developers.redhat.com/blog/2020/04/30/serverless-applications-made-faster-and-simpler-with-openshift-serverless-ga/)

[Using Spring Cloud Functions with OpenShift Serverless](https://slacker.ro/2020/09/01/using-spring-cloud-functions-with-openshift-serverless/)


# Quarkus 

[Quarkus Information](http://saharsh.org/2020/02/04/enter-quarkus/)

[Deploy Quarkus on OpenShift](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.7/html-single/deploying_quarkus_applications_on_red_hat_openshift_container_platform/index)

[Building a Native Executable with Red Hat Build of Quarkus](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/1.7/html-single/building_a_native_executable_with_red_hat_build_of_quarkus/index)

[Quarkus - Container Images](https://quarkus.io/guides/container-image)

# Knative

[Knative](https://knative.dev/)

[kn cli](https://github.com/knative/client)


# Videos

[Red Hat Build of Quarkus 1.7 with OpenShift Serverless](https://www.youtube.com/watch?v=WS99MyypDzM)


# Public Image Repository

[Quay](https://quay.io/repository/)


# OpenShift Commands
```
oc new-project samples

oc project samples

oc new-app --name=valuesdb mysql-ephemeral -p DATABASE_SERVICE_NAME=valuesdb -p MYSQL_ROOT_PASSWORD=password -p MYSQL_USER=valsuser -p MYSQL_PASSWORD=password -p MYSQL_DATABASE=valsdb

oc get pods

oc rsh valuesdb-1-f6z9v bash -c "mysql -uvalsuser -ppassword valsdb"

CREATE TABLE vals (id BIGINT AUTO_INCREMENT PRIMARY KEY, value VARCHAR(255) NOT NULL, date_created TIMESTAMP DEFAULT CURRENT_TIMESTAMP, last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP);

show tables;

oc create secret generic valuesapi-properties --from-literal=SAMPLE_STORAGE_TYPE=persistent --from-literal=QUARKUS_DATASOURCE_URL="jdbc:mysql://valuesdb/valsdb" --from-literal=QUARKUS_DATASOURCE_USERNAME=valsuser --from-literal=QUARKUS_DATASOURCE_PASSWORD=password

docker login -u="kdeif" -p="x" quay.io

docker build -t quay.io/kdeif/sample-quarkus-app:1.0 -f Dockerfile.native  sample-quarkus-app

docker push quay.io/kdeif/sample-quarkus-app:1.0 

kn service create valuesapi --image quay.io/kdeif/sample-quarkus-app:1.0 --env-from secret:valuesapi-properties

```


# Sample Quarkus Application

## Description

This application is a Quarkus implementation of the Values API application from [sample-dotnet-app](https://github.com/saharsh-samples/sample-dotnet-app) repository. The application was created by following the [official guides](https://quarkus.io/get-started/) published by Quarkus. The API exposes the following endpoints:

- `GET /api/values`
- `POST /api/values` (accepts a JSON string in request body)
- `GET /api/values/{id}`
- `PUT /api/values/{id}` (accepts a JSON string in request body)
- `DELETE /api/values/{id}`

The API also comes with the following administrative endpoints:

- `GET /health` (Overall healthcheck)
- `GET /health/live` (Just the liveness healthcheck)
- `GET /health/ready` (Just the readiness healthcheck)
- `GET /metrics` (Prometheus compatible metrics)

In addition to the in memory mode available in the .NET version, this version also adds a persistent mode that stores values created via the API in a MySQL database.

## Build

### JVM image

        docker build -t sample-quarkus-app .

### Native image

        docker build -f Dockerfile.native -t sample-quarkus-app .

## Run Locally

### In memory version

        docker run -d \
            --name sample-quarkus-app \
            -p 8080:8080 \
            sample-quarkus-app

### Persistent version

First run MySQL in a container. `$REPO` below refers to the full path to wherever you cloned this repository.

        export REPO=`pwd` && docker run -d \
            --name valuesdb \
            -e MYSQL_ROOT_PASSWORD=password \
            -v $REPO/database/values.sql:/docker-entrypoint-initdb.d/values.sql:Z \
            -p 3306:3306 \
            mysql:8.0.19

After MySQL is up and running, run the persistent version of API. `$HOST_IP` below refers to the IP address of the machine running these containers. This assumes the API container can reach the MySQL container using the host IP.

        docker run -d \
            --name sample-quarkus-app \
            -e SAMPLE_STORAGE_TYPE=persistent \
            -e QUARKUS_DATASOURCE_URL="jdbc:mysql://$HOST_IP:3306/valsdb" \
            -e QUARKUS_DATASOURCE_USERNAME=valsuser \
            -e QUARKUS_DATASOURCE_PASSWORD=password \
            -p 8080:8080 \
            sample-quarkus-app

## Consume

Following is an example flow to use the web service. **NOTE**: The `host:port` value of `localhost:8080` below is based on the application being deployed using [Run Locally](#run-locally) section above. Change the value as needed when consuming the application that has been deployed using other means.

- Health Check

        curl -i localhost:8080/health

- Prometheus compatible metrics

        curl -i -H "Accept: application/json" \
                localhost:8080/metrics

- Create a value

        curl -i -X POST \
                -H "Content-Type: application/json" \
                -d '"My Original Value"' \
                localhost:8080/api/values

- Fetch all stored values

        curl -i localhost:8080/api/values

- Change value

        curl -i -X PUT \
                -H "Content-Type: application/json" \
                -d '"My Changed Value"' \
                localhost:8080/api/values/1

- Fetch value by ID

        curl -i localhost:8080/api/values/1

- Delete value by ID

        curl -i -X DELETE \
                localhost:8080/api/values/1

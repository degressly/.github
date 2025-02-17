# Degressly

[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/daniyaalk)

---

Degressly is a portmanteau of "Deterministic Regression". Inspired by [opendiffy/diffy](https://github.com/opendiffy/diffy), degressly aims to create a scalable framework for performing regression testing.

Degressly works by running three parallel instances of your code side by side and multicasting all inbound requests to all three instances. The primary and secondary instances run your last known good code, while the candidate instances runs code that is to be tested.
Differences between primary and candidate instances involve noise from non-deterministic sources like random number generation and timestamps, these differences are ignored based on the differences obtained from responses of primary and secondary instances.
This allows you to perform black-box testing of your code without needing to write any tests, or mocking data that is a representative sample of your production environment.

If you want a quick and dirty way to try out degressly with a sample application, check out [degressly/degressly-demo](https://github.com/degressly/degressly-demo).

## Features
* In-depth analysis of differences in behaviour between new and old code. Including:
  * Differences in API Responses.
  * Differences in API requests made by the microservice.
  * Differences in data stored in the datastore.
* Filtering out noise from non-deterministic sources like random number generation and timestamps.
* Native support for replaying requests.
* Support for non-idempotent upstream APIs where each request must be sent once and once only.
  * Support for trapping all upstream calls when replaying requests.

## Architecture
The degressly ecosystem depends on the following repositories:

| Repository               | Description                                                                                                                                                                                                                                                      |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [degressly-core](https://github.com/degressly/degressly-core)       | Core frontend HTTP Proxy. Logs responses from each downstream separately, can be configured to push observations for analysis by degressly-comparator.                                                                                                           |
| [degressly-comparator](https://github.com/degressly/degressly-comparator) | Analyzes observations received from Primary, Secondary and Candidate deployments                                                                                                                                                                                 |
| [degressly-downstream](https://github.com/degressly/degressly-downstream) | Downstream proxy for services that make S2S calls to downstream services, with the ability to accomodate non-idempotent downstreams. Logs requests from each downstream separately, can be configured to push observations for analysis by degressly-comparator. |


![Degressly architecture](images/Degressly.png)

## Getting Started

Clone repository:
```bash
git clone https://github.com/degressly/degressly-core.git
cd degressly-core
```

### Running Docker singleton
```bash
docker build -f Dockerfile -t degressly-core:latest
docker run -p8000:8000 degressly-core:latest
```

### Running with docker-compose

Create .env file:
```bash
touch .env
nano .env
```

Sample `.env` file:
```
primary_host=http://host.docker.internal:9000
secondary_host=http://host.docker.internal:9001
candidate_host=http://host.docker.internal:9002
```
_Note on host names: https://docs.docker.com/desktop/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host_

```bash
docker build -f Dockerfile -t degressly-core:latest
docker compose up
```


### Running full degressly ecosystem with docker-compose:
Clone all repos and build docker images:
```bash
git clone https://github.com/degressly/degressly-core.git
git clone https://github.com/degressly/degressly-comparator.git
git clone https://github.com/degressly/degressly-downstream.git
docker build degressly-core/ -t degressly-core:latest 
docker build degressly-comparator/ -t degressly-comparator:latest 
docker build degressly-downstream/ -t degressly-downstream:latest 
cd degressly-core
```

Create .env file:
```bash
touch .env
```
```
spring_profiles_active=mongo
diff_publisher_bootstrap_servers=kafka:9092
diff_publisher_topic_name=diff_stream
primary_host=http://host.docker.internal:9000
secondary_host=http://host.docker.internal:9001
candidate_host=http://host.docker.internal:9002
MONGO_URL=<mongo_cluster_url>/<mongo_db_name>
MONGO_USERNAME=<mongo_username>
MONGO_PASSWORD=<mongo_password>
MONGO_DBNAME=<mongo_db_name>
```
_Note on host names: https://docs.docker.com/desktop/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host_ 

Run containers
```bash
docker compose --profile full up
```

## Limitations / TODO
_In no particular order:_
* ~~DB layer observation recon (comparator can consume from debezium or similar CDC pipeline).~~
* DB Proxy...?
* Performance regression tracking.
* ~~Dockerization.~~
* Documentation for Replay support.

## Support

If you would like to reach out for support or feature requests, feel free to drop an email at [me@daniyaalkhan.com](mailto:me@daniyaalkhan.com)


# rocky-performance
Part of the 'On Rocky' series. This implementation takes PROMETHEUS monitoring, GFAFANA visualization and JMETER load in to one all inclusive approach to implmenting a Performance Engineering solution that includes Load and monitoring of systems under test. 

A **docker-compose.yaml** is part of this repository for your convience or you may exeucte the following steps outide of Compose. 

## Lets runs some tests
Now we have PROMETHEUS for monitoring, GRAFANA for vistualizations and INFLUXDB for Integration with other apps like Jmeter. 

```
docker network create --driver bridge performance-network
docker run -d --name PROMETHEUS --hostname PROMETHEUS -p 9090:9090 --network performance-network -t subrock/rocky-prometheus
docker run -d --name GRAFANA --hostname GRAFANA -p 9091:3000 --network performance-network -t subrock/rocky-grafana
docker run -d --name INFLUXDB --hostname INFLUXDB -p 8086:8086 -e DOCKER_INFLUXDB_INIT_MODE=setup -e DOCKER_INFLUXDB_INIT_USERNAME=admin -e DOCKER_INFLUXDB_INIT_PASSWORD=admin --network performance-network -t subrock/rocky-influxdb
```
Lets start JMETER.
```
docker run --name CONTROLLER --hostname CONTROLLER --network performance-network -d -t subrock/rocky-jmeter:controller
docker run --name WORKER-1 --hostname WORKER-1 --network performance-network -d -t subrock/rocky-jmeter:worker
```
Optionally, start Node Exporter on INFLUXDB. 
```
docker exec -it INFLUXDB /start_node.sh
```
Wait a good 2 minutes for Exporters to sync. Go to Grafana and click on the Jmeter dashboard. (http://localhost:9091) Then run tests using Jmeter GUI or distributed cluster. You should see test execution results in real-time. 
```
docker exec -it CONTROLLER /usr/local/bin/rocky-jmeter-run install_test_script.jmx
```
## Customizations
This project implments two levels of cusotmization. The first is Persisted data. The second is extensibility beyond base packages. 
### Persisted Data
The grafana.db are prometheus.yml built in to the images so things work out of the box. Volumes are not needed unless you wish to persist data between restarts. (Like when developing.) 
### Extensibility Customizations
An include statement has been added to the prometheus.yml that points to /opt/node/custom_configs. You can modify your discovery computers here. There is a helper node page you can use to view/add/remove target computers. Note; Still working on Endpoints. The Grafana.db has been pre-loaded with dashboards that monitor JMETER and underlying PROMETHEUS, GRAFANA and INFLUXDB nodes. Everything else is out of the box or leverages environment variables. 
## A note about scalling
This approach is an example of how load and performance testing can be delivered as a single solutinon for stakeholders. **This approach should not be used in high load scenarios**. You can improve performance by running containers on sepparate servers and accounting for DNS. Optionally you can piece things togehter by hand on physical servers. For assistance please reference the projects used in this example. 

https://github.com/subrock/rocky-jmeter

https://github.com/subrock/rocky-prometheus

https://github.com/subrock/rocky-grafana

https://github.com/subrock/rocky-influxdb

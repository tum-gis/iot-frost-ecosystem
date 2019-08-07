# Grafana

Grafana is an Open Source application which is being widely used for time series analysis and visualizations. It allows us to query, visualize, alert on and understand time series stored in heterogeneous data sources. Users can very easily create, explore and share different kinds of dashboards.

In this repository, Grafana Dashboards are used to link with the running FROST Server. Grafana already supports a plug-in called *LinkSmart SensorThings Data Source Plugin*. By enabling this plugin, different sensor devices and their real-time observations can very easily be visualized using Grafana Dashboards.

## Installation

Grafana can be easily installed using Docker containers by using the below-mentioned steps. For more details about the installation steps, please visit https://grafana.com/docs/installation/docker/

To run this directly in docker at its simplest, just run

    docker run -p 3000:3000 --name grafana grafana/grafana

The coomand can be explained as follows:

    docker run      - run this container... and build locally if necessary first.
    -p 3000:3000    - connect local port 3000 to the exposed internal port 3000
    --name grafana - give this machine a friendly local name
    grafana/grafana - the image to base it on the latest version

Once Grafana is up and running, you can browse to http://localhost:3000 to interact with the application.

After its installation, it is also important to install additional plug-ins. For the purpose of this repository, we install *LinkSmart SensorThings Data Source Plugin* as follows:

    docker exec -it -u root grafana /bin/bash
    apt-get update
    ./bin/grafana-cli plugins install linksmart-sensorthings-datasource

Other plugins can also be installed in the same ways.

## Setting up Grafana Dashboard for SensorThings API

**Step 1 : Configure the DataSource**

Once the required plugins are installed, the first step is to configure the Data Source. Please select the LinkSmart SensorThings Data Source and provide the base URL of the FROST Server (http://localhost:8080/FROST-Server/v1.0/).

![SensorThings API UML Model](../doc/images/Grafana1.png)

**Step 2 : Create Visualization Charts**

Once the FROST Server is configured, the respective sensor device and observation properties can be visulized in different ways.

![SensorThings API UML Model](../doc/images/Grafana2.png)

## Further Visualization options

For more visualization options in Grafana Dashboard, please visit https://grafana.com/

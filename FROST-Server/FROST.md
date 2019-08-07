# FROST-Server
Due to the complete support of the OGC SensorThings API and Open Source in nature, we prefer to install and work with the FROST Server. The FROST Server uses a PostgreSQL Database to implement the SensorThings API Data Model and allows inserting and querying data using HTTP operations (POST, GET, PATCH, and DELETE). The FROST-Server also supports the MQTT protocol, which is an asynchronous (publish/subscribe) communication protocol. It allows users to subscribe to datastreams they are interested in. Applications then get notified whenever new observations become available.

![SensorThings API UML Model](../doc/images/FROST.png)

## Installation
The FROST-Server can easily be installed using Docker containers by using the below-mentioned steps. For more details about the installation steps, please visit https://github.com/FraunhoferIOSB/FROST-Server

**Step 1: Prepare docker-compose.yaml file**

Compose is a tool for defining and running multi-container Docker applications. With Compose, we use a YAML file to configure the application’s services. Then, with a single command, we can create and start all the services from your configuration. An example docker-compose file also available [here](https://github.com/FraunhoferIOSB/FROST-Server/blob/master/docker-compose.yaml) looks as follows:

version: '2'

    services:
    web:
    image: fraunhoferiosb/frost-server:latest
    environment:
      - serviceRootUrl=http://localhost:8080/FROST-Server
      - http_cors_enable=true
      - http_cors_allowed.origins=*
      - persistence_db_driver=org.postgresql.Driver
      - persistence_db_url=jdbc:postgresql://database:5432/sensorthings
      - persistence_db_username=sensorthings
      - persistence_db_password=ChangeMe
      - persistence_autoUpdateDatabase=true
    ports:
      - 8080:8080
      - 1883:1883
    depends_on:
      - database

    database:
    image: mdillon/postgis:latest
    environment:
      - POSTGRES_DB=sensorthings
      - POSTGRES_USER=sensorthings
      - POSTGRES_PASSWORD=ChangeMe
    volumes:
      - postgis_volume:/var/lib/postgresql/data
    volumes:
    postgis_volume:

The relevant fields can be defined and configured as above.

**Step 2: Save docker-compose.yaml file.**

Once docker-compose file is prepared, you can save it in a folder at your local machine/server.

**Step 3: Run docker-compose.yaml file.**

Please navigate to the folder where the YAML file is saved and run the following command:

    docker-compose up -d

It will deploy the required containers (Tomcat and PostGIS instances) for the FROST Server.

Considering the serviceRootUrl defined in the configuration file is http://localhost:8080/FROST-Server, the base resource path can be tested as http://localhost:8080/FROST-Server/v1.0/

It will return a JSON array of the available SensorThings resource endpoints as illustrated in the [UML diagram](README.md).

## Setting up the FROST-Server
Once the FROST-Server is up and running, the sensor description and observations can be inserted using HTTP operations. The OGC SensorThings API data model is designed in such a way that the static information (e.g. sensor device description, location, and other metadata) and dynamic information (e.g. timeseries observations) are modeled separately. It allows us to insert static information only once and to insert dynamic information in real-time manner.

FROST-Server supports the following HTTP operations:
* GET - to retrieve a resource
* POST - to register a resource
* PATCH - to update a resource
* DELETE - to delete a resource

These operations can be performed by cURL requests for Mac/Linux and the third-party software such as [POSTMAN](https://www.getpostman.com/) and [FME](https://www.safe.com/) for Windows.

### Static Information
**Thing**

A Thing is an object of the physical world (physical Things) or the information world (virtual Things) that is capable of being identified and integrated into communication networks.

Thing is a good starting point to start creating the SensorThings model structure. A Thing has Locations and one or more Datastreams to collect Observations. A minimal Thing can be created without a Location and Datastream and there are options to create a Things with a nested linked Location and Datastream.

Example 1: POST

POSTMAN

    POST http://localhost:8080/FROST-Server/v1.0/Things
    Content-Type application/json


      {
        "name": "my_thing",
        "description": "description_of_the_thing",
        "Locations": [
          {
            "name": "Munich, Germany",
            "encodingType": "application/vnd.geo+json",
            "description": "description_of_location",
            "location": {
              "coordinates":[11.49600700,48.11323500],
              "type":"Point"}
            }
        ]
      }

cURL

    curl -XPOST -H "Content-Type: application/json" -d '{
    "name": "my_thing",
    "description": "description_of_the_thing",
    "Locations": [
      {
        "name": "Munich, Germany",
        "encodingType": "application/vnd.geo+json",
        "description": "description_of_location",
        "location": {
          "coordinates":[11.49600700,48.11323500],
          "type":"Point"}
        }
    ]
    }' "http://localhost:8080/FROST-Server/v1.0/Things"

Example 2: GET

    GET http://localhost:8080/FROST-Server/v1.0/Things

It will list all the registered Things in the FROST-Server. If the registered Thing ID is 1, it can be queried as

    GET http://localhost:8080/FROST-Server/v1.0/Things(1)

Example 3: PATCH

POSTMAN

    PATCH http://localhost:8080/FROST-Server/v1.0/Things(1)
    Content-Type application/json
    {
      "description": "updated_description_of_the_thing"
    }

cURL

      curl -X PATCH -H "Content-Type: application/json" -d '{
      "description": "updated_description_of_the_thing"
      }' "http://localhost:8080/FROST-Server/v1.0/Things(1)"

Example 4: DELETE

POSTMAN

    DELETE http://localhost:8080/FROST-Server/v1.0/Things(1)

cURL 

    curl -X DELETE -H "Content-Type: application/json" -H -d '' "http://localhost:8080/FROST-Server/v1.0/Things(1)"
**ObservationProperty**

An ObservedProperty specifies the phenomenon of an Observation. If the desired ObservedProperty is not registered already, it should be registered in the FROST-Server. In this scenario, if the Thing measures two ObservedProperties: Temperature and Humidity, they can be registered as follows:

Temperature

POSTMAN

    POST http://localhost:8080/FROST-Server/v1.0/ObservedProperties
    Content-Type application/json
    {
      "name": "temperature",
      "description": "https://en.wikipedia.org/wiki/Temperature",
      "definition": "Celsius"
    }

cURL

    curl -XPOST -H "Content-Type: application/json" -d '{
    "name": "temperature",
    "description": "https://en.wikipedia.org/wiki/Temperature",
    "definition": "Celsius"
    }' "http://localhost:8080/FROST-Server/v1.0/ObservedProperties"

Relative Humidity

POSTMAN

    POST http://localhost:8080/FROST-Server/v1.0/ObservedProperties
    Content-Type application/json
    {
      "name": "relative_humidity",
      "description": "https://en.wikipedia.org/wiki/Relative_humidity",
      "definition": "Percent"
    }

cURL

    curl -XPOST -H "Content-Type: application/json" -d '{
    "name": "relative_humidity",
    "description": "https://en.wikipedia.org/wiki/Relative_humidity",
    "definition": "Percent"
    }' "http://localhost:8080/FROST-Server/v1.0/ObservedProperties"

Battery Voltage

POSTMAN

    POST http://localhost:8080/FROST-Server/v1.0/ObservedProperties
    Content-Type application/json
    {
      "name": "battery_voltage",
      "description": "https://en.wikipedia.org/wiki/Voltage",
      "definition": "Voltage"
    }

cURL

      curl -XPOST -H "Content-Type: application/json" -d '{
      "name": "battery_voltage",
      "description": "https://en.wikipedia.org/wiki/Voltage",
      "definition": "Voltage"
    }' "http://localhost:8080/FROST-Server/v1.0/ObservedProperties"

The registered ObservedProperties can be accessed using the GET operations

    GET http://localhost:8080/FROST-Server/v1.0/ObservedProperties - array of all registered ObservedProperties
    GET http://localhost:8080/FROST-Server/v1.0/ObservedProperties(1) - Temperature
    GET http://localhost:8080/FROST-Server/v1.0/ObservedProperties(2) - Relative Humidity
    GET http://localhost:8080/FROST-Server/v1.0/ObservedProperties(3) - Battery Voltage

The PATCH and DELETE operations can be performed in the similar ways as described in the Things section.

**Sensors**

A Sensor in SensorThings API is an instrument that observes a property or phenomenon with the goal of producing an estimate of the value of the property. If the desired Sensor is not registered already, it should be registered in the FROST-Server as follows:

POSTMAN

    POST http://localhost:8080/FROST-Server/v1.0/Sensors
    Content-Type application/json
    {
      "name":"DHT22",
      "encodingType": "application/pdf",
      "metadata": "http://wiki.seeedstudio.com/Grove-Temperature_and_Humidity_Sensor_Pro/",
      "description": "http://wiki.seeedstudio.com/Grove-Temperature_and_Humidity_Sensor_Pro/"
    }

cURL

    curl -XPOST -H "Content-Type: application/json" -d '{
    "name":"DHT22",
    "encodingType": "application/pdf",
    "metadata": "http://wiki.seeedstudio.com/Grove-Temperature_and_Humidity_Sensor_Pro/",
    "description": "http://wiki.seeedstudio.com/Grove-Temperature_and_Humidity_Sensor_Pro/"
    }' "http://localhost:8080/FROST-Server/v1.0/Sensors"


Once the sensor is registered, it can be accessed as follows:

    GET http://localhost:8080/FROST-Server/v1.0/Sensors - array of all registered sensors
    GET http://localhost:8080/FROST-Server/v1.0/ObservedProperties(1) - DHT22

The PATCH and DELETE operations can be performed in the similar ways as described in the Things section.

**Datastreams**

A Datastream groups a collection of Observations measuring the same ObservedProperty and produced by the same Sensor. In order to insert the Datastream, it is important to know the IDs of the Thing, the Sensor, and the ObservedProperty.

For example, the Datastream for the temperature property (ObservedProperty ID:1) being produced by the DHT22 sensor (Sensor ID:1) in the my_thing (Thing ID:1) can be registered as:

POSTMAN

    POST http://localhost:8080/FROST-Server/v1.0/Datastreams
    Content-Type application/json
    {
      "name": "temperature",
      "unitOfMeasurement": {
        "name": "Celsius",
        "symbol": "C",
        "definition": "https://en.wikipedia.org/wiki/Celsius"
      },
      "Thing": {
        "@iot.id": 1
      },
      "description": "This is a datastream for the temperature property from my_thing",
      "Sensor": {
        "@iot.id": 1
      },
      "ObservedProperty": {
    	    "@iot.id": 1
      },
      "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement"
    }

cURL

    curl -XPOST -H "Content-Type: application/json" -d '{
    "name": "temperature",
    "unitOfMeasurement": {
    "name": "Celsius",
    "symbol": "C",
    "definition": "https://en.wikipedia.org/wiki/Celsius"
    },
    "Thing": {
    "@iot.id": 1
    },
    "description": "This is a datastream for the temperature property from my_thing",
    "Sensor": {
    "@iot.id": 1
    },
    "ObservedProperty": {
      "@iot.id": 1
    },
    "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement"
    }' "http://localhost:8080/FROST-Server/v1.0/Datastreams"

Similarly, the Datastream for the relative_humidity property (ObservedProperty ID:2) being produced by the DHT22 sensor (Sensor ID:1) in the my_thing (Thing ID:1) can be registered as:

POSTMAN

    POST http://localhost:8080/FROST-Server/v1.0/Datastreams
    Content-Type application/json
    {
      "name": "humidity",
      "unitOfMeasurement": {
        "name": "Percent",
        "symbol": "%",
        "definition": "https://en.wikipedia.org/wiki/Percentage"
      },
      "Thing": {
        "@iot.id": 1
      },
      "description": "This is a datastream for the relative_humidity property from my_thing",
      "Sensor": {
        "@iot.id": 1
      },
      "ObservedProperty": {
    	    "@iot.id": 2
      },
      "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement"
    }

cURL

    curl -XPOST -H "Content-Type: application/json" -d '{
    "name": "humidity",
    "unitOfMeasurement": {
    "name": "Percent",
    "symbol": "%",
    "definition": "https://en.wikipedia.org/wiki/Percentage"
    },
    "Thing": {
    "@iot.id": 1
    },
    "description": "This is a datastream for the relative_humidity property from my_thing",
    "Sensor": {
    "@iot.id": 1
    },
    "ObservedProperty": {
      "@iot.id": 2
    },
    "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement"
    }' "http://localhost:8080/FROST-Server/v1.0/Datastreams"


And, the Datastream for the battery_voltage property (ObservedProperty ID:3) being produced by the DHT22 sensor (Sensor ID:1) in the my_thing (Thing ID:1) can be registered as:

POSTMAN

        POST http://localhost:8080/FROST-Server/v1.0/Datastreams
        Content-Type application/json
        {
          "name": "battery_voltage",
          "unitOfMeasurement": {
            "name": "Volts",
            "symbol": "V",
            "definition": "https://en.wikipedia.org/wiki/Voltage"
          },
          "Thing": {
            "@iot.id": 1
          },
          "description": "This is a datastream for the relative_humidity property from my_thing",
          "Sensor": {
            "@iot.id": 1
          },
          "ObservedProperty": {
        	    "@iot.id": 3
          },
          "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement"
        }

cURL

    curl -XPOST -H "Content-Type: application/json" -d '{
      "name": "battery_voltage",
      "unitOfMeasurement": {
        "name": "Volts",
        "symbol": "V",
        "definition": "https://en.wikipedia.org/wiki/Voltage"
      },
      "Thing": {
        "@iot.id": 1
      },
      "description": "This is a datastream for the battery_voltage property from my_thing",
      "Sensor": {
        "@iot.id": 1
      },
      "ObservedProperty": {
          "@iot.id": 3
      },
      "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Measurement"
    }' "http://localhost:8080/FROST-Server/v1.0/Datastreams"

Once the sensor is registered, it can be accessed as follows:

    GET http://localhost:8080/FROST-Server/v1.0/Datastreams - array of all registered Datastreams
    GET http://localhost:8080/FROST-Server/v1.0/Datastreams(1) - temperature from my_thing
    GET http://localhost:8080/FROST-Server/v1.0/Datastreams(2) - relative_humidity from my_thing
    GET http://localhost:8080/FROST-Server/v1.0/Datastreams(3) - battery_voltage from my_thing

The PATCH and DELETE operations can be performed in the similar ways as described in the Things section.

### Dynamic information
Once the static information is registered at the FROST-Server with appropriate IDs, the dynamic information can be inserted corresponding to the relevant Datastream ID. For example, the temperature and humidity observations (time-value pairs) from the Iot Platform can be inserted to the FROST-Server against the respective Datastream ID. For this purpose, automated workbenches such as NodeRED can be used.

The timeseries of the temperature recordings of my_thing can be inserted corresponding to the Datastream ID 1 as follows:

    POST http://localhost:8080/FROST-Server/v1.0/Observations
    Content-Type application/json
    {
      "phenomenonTime": "2019-08-05T12:51:14+02:00",
      "result": 22.6,
      "Datastream": {
          "@iot.id": 1
      }
    }

The timeseries of the humidity recordings of my_thing can be inserted corresponding to the Datastream ID 2 as follows:

    POST http://localhost:8080/FROST-Server/v1.0/Observations
    Content-Type application/json
    {
        "phenomenonTime": "2019-08-05T12:51:14+02:00",
        "result": 48,
        "Datastream": {
            "@iot.id": 2
        }
      }

The timeseries of the battery_voltage recordings of my_thing can be inserted corresponding to the Datastream ID 3 as follows:

    POST http://localhost:8080/FROST-Server/v1.0/Observations
    Content-Type application/json
    {
        "phenomenonTime": "2019-08-05T12:51:14+02:00",
        "result": 30,
        "Datastream": {
              "@iot.id": 2
        }
    }

The Observations for each of the Datastream can be accessed as follows:

    GET http://localhost:8080/FROST-Server/v1.0/Datastreams(1)/Observations - temperature from my_thing
    GET http://localhost:8080/FROST-Server/v1.0/Datastreams(2)/Observations - relative_humidity from my_thing
    GET http://localhost:8080/FROST-Server/v1.0/Datastreams(3)/Observations - battery_voltage from my_thing

# Further querying options

For more querying possibilities, please visit https://developers.sensorup.com/docs/#introduction

# Next: [Installation and setting up the Node-RED](../NodeRED/README.md)

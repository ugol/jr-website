---
# Title, summary, and page position.
linktitle: Kafka Connect Source Connector
weight: 550
#icon: book
icon_pack: fas

# Page metadata.
title: Kafka Connect Source Connector
date: '2024-09-02T00:00:00Z'
type: book # Do not modify.
---

## Kafka Connect Source Connector

JR provides a Source Connector for Apache Kafka Connect that is in the same segment of [Kafka Connect Datagen](https://github.com/confluentinc/kafka-connect-datagen)
for generation of synthetic data.

## Configuration

JR Source Connector can be configured with:

Parameter | Description                                                                                                                                                                                                                                     | Default
-|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-
`template` | A valid JR existing template name. Skipped when __embedded_template_ is set. For a list of available templates see: https://jrnd.io/docs/#listing-existing-templates                                                                            | net_device
`embedded_template` | Location of a file or URL, containing a valid custom JR template. This property will take precedence over _template_. File must exist on Kafka Connect Worker nodes.                                                                                    | 
`topic` | destination topic on Kafka                                                                                                                                                                                                                      |
`frequency` | Repeat the creation of a random object every 'frequency' milliseconds.                                                                                                                                                                          | 5000                                                                         
`duration` | Set a time bound to the entire object creation. The duration is calculated starting from the first run and is expressed in milliseconds. At least one run will always been scheduled, regardless of the value for 'duration'. If not set creation will run forever. | -1                                                                                              
`objects` | Number of objects to create at every run.                                                                                                                                                                                                       | 1                                                                                                                                   
`key_field_name` | Name for key field, for example 'ID'. This is an _OPTIONAL_ config, if not set, objects will be created without a key. Skipped when _key_embedded_template_ is set. Value for key will be calculated using JR function _key_, https://jrnd.io/docs/functions/#key |
`key_value_interval_max` | Maximum interval value for key value, for example 150 (0 to key_value_interval_max). Skipped when _key_embedded_template_ is set.                                                                                                               | 100
`key_embedded_template` | Location of a file or URL, containing a valid custom JR template for keys. This property will take precedence over _key_field_name_ and _key_value_interval_max_. File must exist on Kafka Connect Worker nodes.                                        |
`jr_executable_path` | Location for JR executable on workers. If not set, jr executable will be searched using $PATH variable.                                                                                                                                         |
`value.converter` | one between _org.apache.kafka.connect.storage.StringConverter_, _io.confluent.connect.avro.AvroConverter_, _io.confluent.connect.json.JsonSchemaConverter_ or _io.confluent.connect.protobuf.ProtobufConverter_                                 |org.apache.kafka.connect.storage.StringConverter
`value.converter.schema.registry.url` | Only if _value.converter_ is set to _io.confluent.connect.avro.AvroConverter_, _io.confluent.connect.json.JsonSchemaConverter_ or _io.confluent.connect.protobuf.ProtobufConverter_. URL for _Schema Registry._                                 |
`key.converter` | one between _org.apache.kafka.connect.storage.StringConverter_, _io.confluent.connect.avro.AvroConverter_, _io.confluent.connect.json.JsonSchemaConverter_ or _io.confluent.connect.protobuf.ProtobufConverter_                                 |org.apache.kafka.connect.storage.StringConverter
`key.converter.schema.registry.url` | Only if _key.converter_ is set to _io.confluent.connect.avro.AvroConverter_, _io.confluent.connect.json.JsonSchemaConverter_ or _io.confluent.connect.protobuf.ProtobufConverter_. URL for _Schema Registry._                                   |

Following example is for a JR connector job using template _net_device_ and producing 5 new random messages to _net_device_ topic every 5 seconds.

```json
{
    "name" : "jr-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "template" : "net_device",
        "topic": "net_device",
        "frequency" : 5000,
        "objects": 5,
        "tasks.max": 1
    }
}
```

```
kafka-console-consumer --bootstrap-server localhost:9092 --topic net_device --from-beginning --property print.key=true
null	{"VLAN": "BETA","IPV4_SRC_ADDR": "10.1.98.6","IPV4_DST_ADDR": "10.1.185.254","IN_BYTES": 1756,"FIRST_SWITCHED": 1724287965,"LAST_SWITCHED": 1725353374,"L4_SRC_PORT": 80,"L4_DST_PORT": 443,"TCP_FLAGS": 0,"PROTOCOL": 3,"SRC_TOS": 190,"SRC_AS": 1,"DST_AS": 1,"L7_PROTO": 81,"L7_PROTO_NAME": "TCP","L7_PROTO_CATEGORY": "Transport"}
null	{"VLAN": "BETA","IPV4_SRC_ADDR": "10.1.95.4","IPV4_DST_ADDR": "10.1.239.68","IN_BYTES": 1592,"FIRST_SWITCHED": 1722620372,"LAST_SWITCHED": 1724586369,"L4_SRC_PORT": 443,"L4_DST_PORT": 22,"TCP_FLAGS": 0,"PROTOCOL": 0,"SRC_TOS": 165,"SRC_AS": 3,"DST_AS": 1,"L7_PROTO": 443,"L7_PROTO_NAME": "HTTP","L7_PROTO_CATEGORY": "Transport"}
null	{"VLAN": "DELTA","IPV4_SRC_ADDR": "10.1.126.149","IPV4_DST_ADDR": "10.1.219.156","IN_BYTES": 1767,"FIRST_SWITCHED": 1721931269,"LAST_SWITCHED": 1724976862,"L4_SRC_PORT": 631,"L4_DST_PORT": 80,"TCP_FLAGS": 0,"PROTOCOL": 1,"SRC_TOS": 139,"SRC_AS": 0,"DST_AS": 1,"L7_PROTO": 22,"L7_PROTO_NAME": "TCP","L7_PROTO_CATEGORY": "Application"}
```

## Usage of keys

Connector can be configured to create messages having keys. 

In this example a JR connector job for template _user_ will be instantiated and produce 5 new random messages to _user_ topic every 5 seconds, using a message key field named 'guid' set with a random integer value between 0 and 150.

```
{
    "name" : "jr-keys-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "template" : "user",
        "topic": "user",
        "frequency" : 5000,
        "objects": 5,
        "key_field_name": "guid",
        "key_value_interval_max": 150,
        "jr_executable_path": "/usr/bin",
        "tasks.max": 1
    }
}
```
Consume from _user_ topic:

```
kafka-console-consumer --bootstrap-server localhost:9092 --topic user --from-beginning --property print.key=true

{"guid":131}	{  "guid":"131",  "isActive": false,  "balance": "€328.52",  "picture": "http://placehold.it/32x32",  "age": 21,  "eyeColor": "brown",  "name": "Megan Peterson",  "gender": "F",  "company": "Evil Partners",  "work_email": "megan.peterson@evilpartners.com",  "email": "megan.peterson@gmail.com",  "about": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.Fusce elit magna, lobortis nec semper non, aliquam at nisl. Vestibulum elementum suscipit",  "country": "US",  "address": "Tucson, South Street 54, 05602",  "phone_number": "928 59979355",  "mobile": "7109146",  "latitude": 17.1992,  "longitude": -56.3007}
{"guid":70}	{  "guid":"70",  "isActive": true,  "balance": "€633.72",  "picture": "http://placehold.it/32x32",  "age": 24,  "eyeColor": "blue",  "name": "Doris Sanders",  "gender": "F",  "company": "Angels Investors",  "work_email": "doris.sanders@angelsinvestors.com",  "email": "doris.sanders@email.com",  "about": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.Fusce elit magna, lobortis nec semper non, aliquam at nisl. Vestibulum elementum suscipit",  "country": "US",  "address": "Memphis, Oakwood Avenue 8, 02201",  "phone_number": "502 96273890",  "mobile": "7958446",  "latitude": -45.6278,  "longitude": 124.5713}
{"guid":36}	{  "guid":"36",  "isActive": true,  "balance": "€7783.02",  "picture": "http://placehold.it/32x32",  "age": 40,  "eyeColor": "green",  "name": "Sharon Alvarez",  "gender": "F",  "company": "Initech",  "work_email": "sharon.alvarez@initech.com",  "email": "sharon.alvarez@email.com",  "about": "Lorem ipsum dolor sit amet, laoreet ligula. Curabitur id nisl ut Lorem sit amet justo pulvinar aliquet accumsan sit amet",  "country": "US",  "address": "Columbus, Park Place 05, 32301",  "phone_number": "220 06092006",  "mobile": "1856616",  "latitude": 76.7921,  "longitude": 10.1295}
{"guid":9}	{  "guid":"9",  "isActive": false,  "balance": "€6071.06",  "picture": "http://placehold.it/32x32",  "age": 40,  "eyeColor": "brown",  "name": "Michael Jones",  "gender": "M",  "company": "Initech",  "work_email": "michael.jones@initech.com",  "email": "michael.jones@aol.com",  "about": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.Fusce elit magna, lobortis nec semper non, aliquam at nisl. Vestibulum elementum suscipit",  "country": "US",  "address": "Kansas City, South Street 3, 40601",  "phone_number": "689 17290457",  "mobile": "8620336",  "latitude": -61.7961,  "longitude": -167.5185}
{"guid":43}	{  "guid":"43",  "isActive": false,  "balance": "€8298.22",  "picture": "http://placehold.it/32x32",  "age": 38,  "eyeColor": "blue",  "name": "Denise Parker",  "gender": "F",  "company": "Veement Capital Partners",  "work_email": "denise.parker@veementcapitalpartners.com",  "email": "denise.parker@yahoo.com",  "about": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. In ullamcorper non eros eget porta. Aliquam erat volutpat. Mauris molestie lobortis",  "country": "US",  "address": "Charlotte, Queen Street 6, 95814",  "phone_number": "980 95836260",  "mobile": "8203291",  "latitude": 2.5160,  "longitude": 63.2610}
```

## Usage of duration

Connector can be configured to run for a duration of time. 

In this example a JR connector job for template _marketing_campaign_finance_ will be instantiated and produce 5 new random messages to _users_ topic every 10 seconds for a total duration of 30 seconds.

```
{
    "name" : "jr-duration-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "template" : "marketing_campaign_finance",
        "topic": "marketing_campaign_finance",
        "frequency" : 10000,
        "duration" : 30000,
        "objects": 5,
        "tasks.max": 1
    }
}
```

Consume from _marketing_campaign_finance_ topic:

```
kafka-console-consumer --bootstrap-server localhost:9092 --topic marketing_campaign_finance --from-beginning

{  "time": 1610894253695,  "candidate_id": "A3272238",  "party_affiliation": "DEM",  "contribution": 1684}
{  "time": 1614389092497,  "candidate_id": "G8822487",  "party_affiliation": "DEM",  "contribution": 3166}
{  "time": 1600022334958,  "candidate_id": "G5165512",  "party_affiliation": "REP",  "contribution": 2933}
{  "time": 1594525458073,  "candidate_id": "X2839265",  "party_affiliation": "DEM",  "contribution": 824}
{  "time": 1606508742842,  "candidate_id": "T5688428",  "party_affiliation": "IND",  "contribution": 966}
{  "time": 1614055215125,  "candidate_id": "E4299542",  "party_affiliation": "DEM",  "contribution": 1240}
{  "time": 1610035678542,  "candidate_id": "H9769974",  "party_affiliation": "IND",  "contribution": 1793}
{  "time": 1609662702352,  "candidate_id": "S2314618",  "party_affiliation": "DEM",  "contribution": 1531}
{  "time": 1601632523200,  "candidate_id": "A8111647",  "party_affiliation": "IND",  "contribution": 2650}
{  "time": 1612493464065,  "candidate_id": "B1157343",  "party_affiliation": "DEM",  "contribution": 628}
{  "time": 1617678398100,  "candidate_id": "S7362235",  "party_affiliation": "REP",  "contribution": 3405}
{  "time": 1608939902703,  "candidate_id": "N9165865",  "party_affiliation": "REP",  "contribution": 1909}
{  "time": 1599100684111,  "candidate_id": "B2399959",  "party_affiliation": "REP",  "contribution": 1472}
{  "time": 1606312277382,  "candidate_id": "J1118736",  "party_affiliation": "IND",  "contribution": 1156}
{  "time": 1589668105856,  "candidate_id": "Q8211968",  "party_affiliation": "REP",  "contribution": 3457}

Processed a total of 15 messages
```

## Schema Registry: Avro

Connector can be configured to produce objects serialized using Avro. 

In this example a JR connector job for template _store_ will be instantiated and produce 5 new random messages to _store_ topic every 5 seconds, using the _Confluent Schema Registry_ to register the _Avro_ schema.

```
{
    "name" : "jr-avro-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "template" : "store",
        "topic": "store",
        "frequency" : 5000,
        "objects": 5,
        "value.converter": "io.confluent.connect.avro.AvroConverter",
        "value.converter.schema.registry.url": "http://schema-registry:8081",
        "tasks.max": 1
    }
}
```

Consume from _store_ topic:

```
kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic store --from-beginning --property schema.registry.url=http://localhost:8081

{"store_id":1,"city":"Minneapolis","state":"AR"}
{"store_id":2,"city":"Baltimore","state":"LA"}
{"store_id":3,"city":"Chicago","state":"IL"}
{"store_id":4,"city":"Chicago","state":"MN"}
{"store_id":5,"city":"Washington","state":"OH"}
```

Show the _Avro_ schema registered:

```
curl -v http://localhost:8081/subjects/store-value/versions/1/schema
< HTTP/1.1 200 OK
< Content-Type: application/vnd.schemaregistry.v1+json


{"type":"record","name":"storeRecord","fields":[{"name":"store_id","type":"int"},{"name":"city","type":"string"},{"name":"state","type":"string"}],"connect.name":"storeRecord"}
```

## Schema Registry: Json schema

Connector can be configured to produce objects serialized using JsonSchema. 

In this example a JR connector job for template _payment_credit_card_ will be instantiated and produce 5 new random messages to _payment_credit_card_ topic every 5 seconds, using the _Confluent Schema Registry_ to register the _Json_ schema.

```
{
    "name" : "jr-jsonschema-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "template" : "payment_credit_card",
        "topic": "payment_credit_card",
        "frequency" : 5000,
        "objects": 5,
        "value.converter": "io.confluent.connect.json.JsonSchemaConverter",
        "value.converter.schema.registry.url": "http://schema-registry:8081",
        "tasks.max": 1
    }
}
```

Consume from _payment_credit_card_ topic:

```
kafka-json-schema-console-consumer --bootstrap-server localhost:9092 --topic payment_credit_card --from-beginning --property schema.registry.url=http://localhost:8081

{"cvv":"070","card_number":"4086489674117803","expiration_date":"10/24","card_id":1.0}
{"cvv":"505","card_number":"346185299753204","expiration_date":"09/27","card_id":2.0}
{"cvv":"690","card_number":"47606709930001","expiration_date":"12/24","card_id":3.0}
{"cvv":"706","card_number":"4936815806226074","expiration_date":"08/24","card_id":4.0}
{"cvv":"855","card_number":"4782025916077384","expiration_date":"09/22","card_id":5.0}
```

Show the _Json_ schema registered:

```
curl -v http://localhost:8081/subjects/payment_credit_card-value/versions/1/schema
< HTTP/1.1 200 OK
< Content-Type: application/vnd.schemaregistry.v1+json


{"type":"object","properties":{"cvv":{"type":"string","connect.index":2},"card_number":{"type":"string","connect.index":1},"expiration_date":{"type":"string","connect.index":3},"card_id":{"type":"number","connect.index":0,"connect.type":"float64"}}}
```

## Schema Registry: Protobuf

Connector can be configured to produce objects serialized using Protobuf. 

In this example a JR connector job for template _shopping_rating_ will be instantiated and produce 5 new random messages to _shopping_rating_ topic every 5 seconds, using the _Confluent Schema Registry_ to register the _Protobuf_ schema.

```
{
    "name" : "jr-protobuf-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "template" : "shopping_rating",
        "topic": "shopping_rating",
        "frequency" : 5000,
        "objects": 5,
        "value.converter": "io.confluent.connect.protobuf.ProtobufConverter",
        "value.converter.schema.registry.url": "http://schema-registry:8081",
        "tasks.max": 1
    }
}
```

Consume from _shopping_rating_ topic:

```
kafka-protobuf-console-consumer --bootstrap-server localhost:9092 --topic shopping_rating --from-beginning --property schema.registry.url=http://localhost:8081

{"ratingId":1,"userId":0,"stars":2,"routeId":2348,"ratingTime":1,"channel":"iOS-test","message":"thank you for the most friendly,helpful experience today at your new lounge"}
{"ratingId":2,"userId":0,"stars":1,"routeId":6729,"ratingTime":13,"channel":"iOS","message":"why is it so difficult to keep the bathrooms clean ?"}
{"ratingId":3,"userId":0,"stars":3,"routeId":1137,"ratingTime":25,"channel":"ios","message":"Surprisingly good,maybe you are getting your mojo back at long last!"}
{"ratingId":4,"userId":0,"stars":2,"routeId":7306,"ratingTime":37,"channel":"android","message":"worst. flight. ever. #neveragain"}
{"ratingId":5,"userId":0,"stars":3,"routeId":2982,"ratingTime":49,"channel":"android","message":"meh"}
```

Show the _Protobuf_ schema registered:

```
curl -v http://localhost:8081/subjects/shopping_rating-value/versions/1/schema
< HTTP/1.1 200 OK
< Content-Type: application/vnd.schemaregistry.v1+json


syntax = "proto3";

message shopping_rating {
  int32 rating_id = 1;
  int32 user_id = 2;
  int32 stars = 3;
  int32 route_id = 4;
  int32 rating_time = 5;
  string channel = 6;
  string message = 7;
}
```

## Custom templates

Connector can be configured using a custom template for keys and values. 

In this example a JR connector job with a custom template for values will be instantiated and produce 5 new random messages to _customer_ topic every 5 seconds, using the _Confluent Schema Registry_ to register the _Avro_ schema.

Template definition is loaded from file _/tmp/customer-template.json_ existing on Kafka Connect Worker nodes.

Definition for _customer-template.json_:

```
{
    "customer_id": "{{uuid}}",
    "first_name": "{{name}}",
    "last_name": "{{surname}}",
    "email": "{{email}}",
    "phone_number": "{{phone}}",
    "street_address": "{{city}}, {{street}} {{building 2}}, {{zip}}",
    "state": "{{state}}",
    "zip_code": "{{zip}}",
    "country": "United States",
    "country_code": "US"
}
```

Connector job:

```
{
    "name" : "jr-avro-custom-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "embedded_template" : "/tmp/customer-template.json",
        "topic": "customer",
        "frequency" : 5000,
        "objects": 5,
        "value.converter": "io.confluent.connect.avro.AvroConverter",
        "value.converter.schema.registry.url": "http://schema-registry:8081",
        "tasks.max": 1
    }
}
```

Show the _Avro_ schema registered for value:

```
curl -v http://localhost:8081/subjects/customer-value/versions/1/schema
< HTTP/1.1 200 OK
< Content-Type: application/vnd.schemaregistry.v1+json


{"type":"record","name":"recordvalueRecord","fields":[{"name":"customer_id","type":"string"},{"name":"first_name","type":"string"},{"name":"last_name","type":"string"},{"name":"email","type":"string"},{"name":"phone_number","type":"string"},{"name":"street_address","type":"string"},{"name":"state","type":"string"},{"name":"zip_code","type":"string"},{"name":"country","type":"string"},{"name":"country_code","type":"string"}],"connect.name":"recordvalueRecord"}
```

Consume from _customer_ topic:

```
kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic customer --from-beginning --property schema.registry.url=http://localhost:8081

{"customer_id":"6775933f-89c2-43b0-9eaf-e52e5f23293c","first_name":"Cynthia","last_name":"Foster","email":"cynthia.foster@hotmail.com","phone_number":"623 27678252","street_address":"Louisville, Cedar Lane 99, 21401","state":"Massachusetts","zip_code":"21401","country":"United States","country_code":"US"}
{"customer_id":"a15f891e-a3e7-4720-bf59-28202596c667","first_name":"Zachary","last_name":"Harris","email":"zachary.harris@aol.com","phone_number":"747 95821702","street_address":"Austin, River Road 8, 99801","state":"Illinois","zip_code":"99801","country":"United States","country_code":"US"}
{"customer_id":"8906111f-d6d3-4115-bd1a-3e231e3caaa2","first_name":"Julie","last_name":"Long","email":"julie.long@email.com","phone_number":"718 08720661","street_address":"Raleigh, Peachtree Street 43, 58501","state":"Georgia","zip_code":"58501","country":"United States","country_code":"US"}
{"customer_id":"9864ef53-eadf-4012-9cd0-c79e755169df","first_name":"Bryan","last_name":"Wilson","email":"bryan.wilson@mac.com","phone_number":"984 61669636","street_address":"San Antonio, Juniper Drive 23, 17101","state":"Illinois","zip_code":"17101","country":"United States","country_code":"US"}
{"customer_id":"a57911e5-dc9e-4da4-b280-1c0b0143538e","first_name":"Charles","last_name":"Thompson","email":"charles.thompson@gmail.com","phone_number":"726 39040449","street_address":"Richmond, Hillcrest Road 6, 43215","state":"Indiana","zip_code":"43215","country":"United States","country_code":"US"}
```

In this second example a JR connector job with a custom template for values will be instantiated and produce 5 new random messages to _customer_ topic every 5 seconds, using the _Confluent Schema Registry_ to register the _Avro_ schema.

Template definition is loaded from URL _http://web/job-template.json_ .

Definition for _job-template.json_:

```
{
  "job_title": "{{randoms "Software Engineer|Data Scientist|DevOps Engineer|Product Manager|UI/UX Designer"}}",
  "job_department": "{{randoms "Engineering|Data Science|Operations|Product|Design"}}",
  "job_location": "{{randoms "New York|San Francisco|Remote|London|Berlin"}}",
  "job_description": "{{randoms "Join our innovative team to build cutting-edge software solutions|Lead projects in developing next-gen data products|Help design and implement cloud infrastructure for our services|Collaborate with cross-functional teams to enhance our product line|Design user-centered interfaces for our web and mobile applications"}}",
  "required_skills": [
    "{{randoms "Python|Java|JavaScript|AWS|Docker"}}",
    "{{randoms "Agile methodologies|SQL|React|Node.js|Machine Learning"}}",
    "{{randoms "Kubernetes|Figma|Project Management|Data Visualization|Microservices Architecture"}}"],
  "salary": "{{randoms "$80,000 - $100,000|$100,000 - $120,000|$120,000 - $150,000"}}",
  "experience_level": "{{randoms "Entry Level|Mid Level|Senior Level"}}"
}
```

Connector job:

```
{
    "name" : "jr-avro-job-template-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "embedded_template" : "http://web/job-template.json",
        "topic": "jobs",
        "frequency" : 5000,
        "objects": 5,
        "value.converter": "io.confluent.connect.avro.AvroConverter",
        "value.converter.schema.registry.url": "http://schema-registry:8081",
        "tasks.max": 1
    }
}
```

Show the _Avro_ schema registered for value:

```
curl -v http://localhost:8081/subjects/jobs-value/versions/1/schema
< HTTP/1.1 200 OK
< Content-Type: application/vnd.schemaregistry.v1+json


{"type":"record","name":"recordvalueRecord","fields":[{"name":"job_title","type":"string"},{"name":"job_department","type":"string"},{"name":"job_location","type":"string"},{"name":"job_description","type":"string"},{"name":"required_skills","type":{"type":"array","items":"string"}},{"name":"salary","type":"string"},{"name":"experience_level","type":"string"}],"connect.name":"recordvalueRecord"}
```

Consume from _jobs_ topic:

```
kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic jobs --from-beginning --property schema.registry.url=http://localhost:8081

{"job_title":"UI/UX Designer","job_department":"Data Science","job_location":"Remote","job_description":"Collaborate with cross-functional teams to enhance our product line","required_skills":["JavaScript","Node.js","Figma"],"salary":"$120,000 - $150,000","experience_level":"Senior Level"}
{"job_title":"DevOps Engineer","job_department":"Design","job_location":"New York","job_description":"Help design and implement cloud infrastructure for our services","required_skills":["JavaScript","Machine Learning","Figma"],"salary":"$120,000 - $150,000","experience_level":"Senior Level"}
{"job_title":"DevOps Engineer","job_department":"Product","job_location":"London","job_description":"Lead projects in developing next-gen data products","required_skills":["JavaScript","Node.js","Microservices Architecture"],"salary":"$100,000 - $120,000","experience_level":"Mid Level"}
{"job_title":"Product Manager","job_department":"Engineering","job_location":"Remote","job_description":"Collaborate with cross-functional teams to enhance our product line","required_skills":["Java","Machine Learning","Project Management"],"salary":"$80,000 - $100,000","experience_level":"Mid Level"}
{"job_title":"UI/UX Designer","job_department":"Product","job_location":"London","job_description":"Design user-centered interfaces for our web and mobile applications","required_skills":["JavaScript","Machine Learning","Data Visualization"],"salary":"$80,000 - $100,000","experience_level":"Entry Level"}
```

### Custom templates for keys

In this example a JR connector job using a custom template for values will be instantiated and produce 5 new random messages to _customer_full_ topic every 5 seconds, using the _Confluent Schema Registry_ to register the _Avro_ schema. Message keys are also created using a custom template, using the _Confluent Schema Registry_ to register the _Avro_ schema.

Template definition is loaded from file _/tmp/customer-template.json_ existing on Kafka Connect Worker nodes.

Key Template definition is loaded from file _/tmp/key-customer-template.json_ existing on Kafka Connect Worker nodes.

Definition for _customer-template.json_:

```
{
    "customer_id": "{{uuid}}",
    "first_name": "{{name}}",
    "last_name": "{{surname}}",
    "email": "{{email}}",
    "phone_number": "{{phone}}",
    "street_address": "{{city}}, {{street}} {{building 2}}, {{zip}}",
    "state": "{{state}}",
    "zip_code": "{{zip}}",
    "country": "United States",
    "country_code": "US"
}
```

Definition for _key-customer-template.json_:

```
{
  "customer_id": "{{uuid}}",
  "last_name": "{{surname}}"
}
```

Connector job:

```
{
    "name" : "jr-avro-custom-full-quickstart",
    "config": {
        "connector.class" : "io.jrnd.kafka.connect.connector.JRSourceConnector",
        "embedded_template" : "/tmp/customer-template.json",
        "key_embedded_template" : "/tmp/key-customer-template.json",
        "topic": "customer_full",
        "frequency" : 5000,
        "objects": 5,
        "key.converter": "io.confluent.connect.avro.AvroConverter",
        "key.converter.schema.registry.url": "http://schema-registry:8081",
        "value.converter": "io.confluent.connect.avro.AvroConverter",
        "value.converter.schema.registry.url": "http://schema-registry:8081",
        "tasks.max": 1
    }
}
```

Show the _Avro_ schema registered for value:

```
curl -v http://localhost:8081/subjects/customer_full-value/versions/1/schema
< HTTP/1.1 200 OK
< Content-Type: application/vnd.schemaregistry.v1+json


{"type":"record","name":"recordvalueRecord","fields":[{"name":"customer_id","type":"string"},{"name":"first_name","type":"string"},{"name":"last_name","type":"string"},{"name":"email","type":"string"},{"name":"phone_number","type":"string"},{"name":"street_address","type":"string"},{"name":"state","type":"string"},{"name":"zip_code","type":"string"},{"name":"country","type":"string"},{"name":"country_code","type":"string"}],"connect.name":"recordvalueRecord"}
```

Show the _Avro_ schema registered for key:

```
curl -v http://localhost:8081/subjects/customer_full-key/versions/1/schema
< HTTP/1.1 200 OK
< Content-Type: application/vnd.schemaregistry.v1+json


{"type":"record","name":"recordkeyRecord","fields":[{"name":"customer_id","type":"string"},{"name":"last_name","type":"string"}],"connect.name":"recordkeyRecord"}
```

Consume from _customer_full_ topic:

```
kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic customer_full --from-beginning --property schema.registry.url=http://localhost:8081 --property print.key=true

{"customer_id":"e3beafc6-4916-4da4-a624-a8b80f689f51","last_name":"Gonzalez"}	{"customer_id":"e3beafc6-4916-4da4-a624-a8b80f689f51","first_name":"Linda","last_name":"Gonzalez","email":"linda.james@yahoo.com","phone_number":"414 91888379","street_address":"Orlando, Cypress Street 14, 03301","state":"Maryland","zip_code":"03301","country":"United States","country_code":"US"}
{"customer_id":"351ccdd0-c4da-4361-a0fd-2b88c2c28599","last_name":"Gonzalez"}	{"customer_id":"351ccdd0-c4da-4361-a0fd-2b88c2c28599","first_name":"Samantha","last_name":"Gonzalez","email":"samantha.gutierrez@hotmail.com","phone_number":"405 66116008","street_address":"Tampa, River Street 3, 84111","state":"Maine","zip_code":"84111","country":"United States","country_code":"US"}
{"customer_id":"56b99aef-0764-4cc2-9cc6-82d0f9d0d97a","last_name":"Taylor"}	{"customer_id":"56b99aef-0764-4cc2-9cc6-82d0f9d0d97a","first_name":"Frances","last_name":"Taylor","email":"frances.reed@yahoo.com","phone_number":"813 51891158","street_address":"Washington, Franklin Avenue 3, 23219","state":"Arizona","zip_code":"23219","country":"United States","country_code":"US"}
{"customer_id":"ddea0697-1218-40f0-81e8-3d56e324f5c6","last_name":"Baker"}	{"customer_id":"ddea0697-1218-40f0-81e8-3d56e324f5c6","first_name":"Wayne","last_name":"Baker","email":"wayne.brooks@aol.com","phone_number":"571 29789830","street_address":"Richmond, River Street 04, 43215","state":"Iowa","zip_code":"43215","country":"United States","country_code":"US"}
{"customer_id":"0a0ea230-035e-441f-b969-9c6ad5d6f91b","last_name":"Campbell"}	{"customer_id":"0a0ea230-035e-441f-b969-9c6ad5d6f91b","first_name":"Donald","last_name":"Campbell","email":"donald.carter@aol.com","phone_number":"804 33076187","street_address":"Dallas, Orange Street 43, 30303","state":"Wyoming","zip_code":"30303","country":"United States","country_code":"US"}
```

## Additional info 

Additional details are listed in the [official repository](https://github.com/jrnd-io/jr-kafka-connect-source).

JR Source Connector is available on Confluent Hub: https://www.confluent.io/hub/jrndio/jr-source-connector

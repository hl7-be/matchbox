# matchbox 

matchbox is a FHIR server based on the [hapifhir/hapi-fhir-jpaserver-starter](https://github.com/hapifhir/hapi-fhir-jpaserver-starter) with the following features included:
- SDC (Structured Data Capture) [extraction](https://build.fhir.org/ig/HL7/sdc/extraction.html#map-extract) support based on the FHIR Mapping language and [Questionnaire/$extract](http://build.fhir.org/ig/HL7/sdc/OperationDefinition-QuestionnaireResponse-extract.html)
- FHIR Mapping Language endpoints for creation of StructureMaps and support for the [StructureMap/$transform](https://www.hl7.org/fhir/operation-structuremap-transform.html) operation
- Ability to (pre-)load FHIR implementation guides from the package server for conformance resources (StructureMap, Questionnaire, CodeSystem, ValueSet, ConceptMap, NamingSystem, StructureDefinition). The "with-preload" subfolder contains an example with the implementation guides provided for the [public test server](https://test.ahdis.ch/matchbox/fhir).
- validation support: [server]/$validate for checking FHIR resources conforming to the loaded implementation guides
- Ability to serve local static content e.g. a FHIR Questionnaire Renderer or [other SDC-related tools](https://confluence.hl7.org/display/FHIRI/SDC+Implementations)

#TL;DR
To run as a Docker image with static content hosted from a bind volume:
```powershell
mvn package -DskipTests
docker build -t matchbox .
docker run -d --name matchbox -p 8080:8080 matchbox
docker run -d --name matchbox -p 8080:8080 -e "hapi.fhir.staticLocation: file:/static/" -v $PWD/static:/static matchbox
```
(staticLocation is the folder in the container that contains the static content)


FHIR server will then be accessible at http://localhost:8080/matchbox/fhir/metadata. 
Static content will be accessible at http://localhost:8080/matchbox/static. A sample static HTML page at http://localhost:8080/matchbox/static/index.html includes a questionnaire, the mapping and the transformation.

TO DO: see the [issue board](https://github.com/hl7-be/matchbox/projects/1)




## containers

The docker file will create a docker image with no preloaded implementation guides. A list of implementation guides to load can be passed as config-map.
There is another docker file which will create an image with fixed configuration and preloaded implementation guides.  That docker image does not need to download the implementation guides afterwards.


## Prerequisites

- [This project](https://github.com/ahdis/matchbox) checked out. You may wish to create a GitHub Fork of the project and check that out instead so that you can customize the project and save the results to GitHub. Check out the main branch (master is kept in sync with [hapi-fhir-jpaserver-starter](https://github.com/hapifhir/hapi-fhir-jpaserver-starter)
- Oracle Java (JDK) installed: Minimum JDK8 or newer.
- Apache Maven build tool (newest version)

## Running locally

The easiest way to run this server entirely depends on your environment requirements. At least, the following 4 ways are supported:

### Using spring-boot

With no implementation guide:
```bash
mvn clean install -DskipTests spring-boot:run
```
Load example implementation guides:
```bash
mvn clean install -DskipTests spring-boot:run -Dspring-boot.run.arguments=--spring.config.additional-location=file:with-preload/application.yaml
```
or
```
java -Dspring.config.additional-location=file:with-preload/application.yaml -jar target/matchbox.jar
```

```
mvn clean install -DskipTests spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"
```


Then, browse to the following link to use the server:

[http://localhost:8080/matchbox/](http://localhost:8080/matchbox/)

## Building with Docker

### Configurable base image:

```bash
mvn package -DskipTests
docker build -t matchbox .
docker run -d --name matchbox -p 8080:8080 matchbox
```
Server will then be accessible at http://localhost:8080/matchbox/fhir/metadata. 

To dynamically configure run in a kubernetes environment and add a kubernetes config map that provides /config/application.yaml file with implementation guide list like in "with-preload/application.yaml" 

### Image with preloaded implementation guides

After building the base image:
```bash
cd with-preload
docker build -t matchbox-swissepr .
docker run -d --name matchbox-swissepr -p 8080:8080 matchbox-swissepr
```

### making container available
```
docker tag matchbox eu.gcr.io/fhir-ch/matchbox:v141

docker push eu.gcr.io/fhir-ch/matchbox:v141
docker push eu.gcr.io/fhir-ch/matchbox-swissepr:v140
```

API
===

Use VSCode, REST Client to work with the API:
- [FHIR Mapping language](fml.http)
- [Load Implementation Guide onto server](ig.http)
- [Validation examples](validation-igexamples.http)
- SDC TODO


Kubernetes
==========

kubectl cp matchbox-test-0:fhir.logdir_IS_UNDEFINED ./fhir.logdir/
# Lantern

This is just an adaptation to make it easy to deploy the server with apps


## Build
mvn package -DskipTests
docker build -t lantern .

# Push
docker tag lantern docker.io/costateixeira/lantern
docker push docker.io/costateixeira/lantern

# Use
docker run -d --name lantern -v $PWD/apps:/apps -p 8080:8080 costateixeira/lantern

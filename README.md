# CLO835-Assignment1-Application

## Install the required MySQL package
```
sudo apt-get update -y
sudo apt-get install mysql-client -y
```
## Running application locally
```
pip3 install -r requirements.txt
sudo python3 app.py
```
## Building and running 2 tier web application locally
### Building mysql docker image 
```
docker build -t jg_db -f Dockerfile_mysql .
```

### Building application docker image 
```
docker build -t jg_app -f Dockerfile .
```

### Running mysql
```
docker run -d -e MYSQL_ROOT_PASSWORD=wordpass jg_db
```

### Get the IP of the database and export it as DBHOST variable
```
docker inspect <container_id>
```

### Example when running DB runs as a docker container and app is running locally
```
export DBHOST=172.17.0.2
export DBPORT=3306
```
### Other Environment Variables
```
export DBUSER=root
export DATABASE=employees
export DBPWD=wordpass
export APP_COLOR=blue
```
### Run the application, make sure it is visible in the browser
```
docker run -p 8080:8080  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e  DBUSER=$DBUSER -e DBPWD=$DBPWD  jg_app
```

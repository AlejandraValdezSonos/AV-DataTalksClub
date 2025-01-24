# Homework: Week 1

## Question 1. Understanding docker first run
Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash. What's the version of pip in the image?

Pull docker image 
``` bash
docker pull python:3.12.8
``` 
Run the container in interactive mode:
``` bash
docker run -it --entrypoint bash python:3.12.8
```
Confirm python version:
```bash
python --version
```
output: ``` Python 3.12.8```

Check the ```pip``` version inside the container I ran:
``` bash
pip --version
```
output: 
```pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)```

## Question 2: Understanding Docker Networking and Docker-compose
Given the following ```docker-compose.yaml```, what is the ```hostname``` and ```port``` that pgadmin should use to connect to the postgres database?
```yml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```
Answer: 
hostname: db
port: 5432

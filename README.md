# Postman Collections Using Docker

Run Postman collections against the REST API.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

## Prerequisites

To run this project, you need to have:

```
  Docker for Desktop
```
You can download here: https://www.docker.com/


```
  Linux as OS
```
Running `Linux` is mandatory. In order to access the host network from a Docker container i used in `docker-compose.yml` the parameter `network_mode: host`. Unfortunately the  [host networking](https://docs.docker.com/network/host/) driver only works on Linux hosts. For Docker Desktop for Mac or Windows you have to do a [workaround](https://biancatamayo.me/blog/2017/11/03/docker-add-host-ip/).  


```
  cd root_directory_of_RESTAPI_project
```
We navigate to the root directory where the RESTAPI project is located. Not to the postman_tests directory.


```
  sudo docker system prune --volumes
```
We want our postman collection to run against the initial state of the db.
We make sure we delete all volumes and db leftovers.


```
  sudo docker-compose up --build
```
We bring up the RESTAPI.

Now we are ready ...

## Installing and Running

* `git clone git@github.com:SoutisV/postman_tests.git`
* `cd root_directory_of_project`
* `sudo docker-compose up --build`

The `sudo docker-compose up --build` will bring run the postman collection against the RESTAPI.

## Building details

My current `Dockerfile` looks like this, I needed `nodejs` so I have used a version of the `node` base Image and then installed the two modules 'globally' via `npm`. This file also creates a new `WORKDIR` and specifies an `ENTRYPOINT`.  

```bash
FROM node:10.11.0-alpine

RUN npm install -g newman newman-reporter-html

WORKDIR /etc/newman

ENTRYPOINT ["newman"]
```

I also created a `docker-compose.yml` file to help make running the collection easier.

```yml
version: "3"
services:
  postman_tests:
    build: .
    image: postman_tests
    network_mode: host
    command:
        run restful_api_tests.json
        -r html,cli
        --reporter-html-export reports/restful_api_tests_run.html
    volumes:
      - ./src:/etc/newman
```

## Collection Run Output On The Command Line
The default output if you're using `newman` locally or using the Docker Image, is on the CLI - This will give you a simple breakdown of what happened during the collection run.

[Collection Output](https://github.com/SoutisV/postman_tests/blob/master/CLI.png)

## Collection Run HTML Reports

The reason I'm not using the Official Image is to get a custom HTML report from the collection run - For this I'm using the `newman-html-reporter` module, in order to tell `newman` to use this I've added a couple of arguments to the `command` property in the `docker-compose.yml` file. First, we need to tell it we want to use an additional reporter, this is done using the `-r cli,html` flag. This alone would be enough to create the report and save the to the root directory but I wanted it to be placed in a particular directory. For this I've used the `--reporter-html-export` flag and then specified a filename and a location `reports/Restful_Booker_Test_Run.html`.

After the run, the new HTML report is added to the `.src/reports` directory.

## Running The Collection

To run this collection from the command line, assuming you have Docker running on your flavour of OS, type the following:

```bash
docker-compose up
```

This _should_ pull the `node` image and install all the components you need, before running through the tests.

## Expected results

In order to successfully run the collection `restful_api_tests.json` located in `src` we must have the RESTAPI running and also have it in the initial state.
Otherwise the results would not be successful.
So, always regarding the RESTAPI:

```
  sudo docker system prune --volumes
```
and then
```
  sudo docker-compose up --build
```
And then regarding the postman_tests:
```bash
docker-compose up
```
If you run consecutively the collection against the RESTAPI, some tests would [fail](https://github.com/SoutisV/postman_tests/blob/master/fail_run.png).

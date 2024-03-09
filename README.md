# student-list 
This repo is a simple application to list student with a webserver (PHP) and API (Flask)

![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)


------------


## Context


*POZOS*  is an IT company located in France and develops software for High School.

The innovation department want to disrupt the existing infrastructure to ensure that

it can be scalable, easily deployed with a maximum of automation.

POZOS wants to build a **POC** to show how docker can help you and how much this technology is efficient.

For this POC, POZOS will give you an application and want you to build a "decouple" infrastructure based on **Docker**.

Currently, the application is running on a single server with any scalability and any high availability.

When POZOS needs to deploy a new release, every time some goes wrong.

In conclusion, POZOS needs agility on its software farm.

## Infrastructure

POZOS recommends to use centos7.6 OS because it's the most used in the company.

As recommended by the customer,a Centos 7 machine was provisioned using the Vagrant tool.


## Application


The application is named "*student_list*", this application is very basic and enables POZOS to show the list of the student with their age.

student_list has two modules:

- the first module is a REST API (with basic authentication needed) who send the desire list of the student based on JSON file
- The second module is a web app written in HTML + PHP who enable end-user to get a list of students

Now it is time to explain you each file's role:

- docker-compose.yml: to launch the application (API and web app)
- Dockerfile: the file that will be used to build the API image (details will be given)
- requirements.txt: contains all the packages to be installed to run the application
- student_age.json: contain student name with age on JSON format
- student_age.py: contains the source code of the API in python
- index.php: PHP  page where end-user will be connected to interact with the service to - list students with age. 

## Build and test 

1- create images docker:

 docker build -t student-list-img:v1 .
 
 docker images
 
 ![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/c40030b8-5fa1-41f1-a01a-92c5c8c56828)


2-create network "bridge type"

docker network create student-list-network

docker network ls


![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/779fda78-6784-409d-b026-73cbffa3cef2)


3-create container running the API flask python using persistent volume, network brige type on the port 5000

docker run --rm -d --name=student_list_api -p 5000:5000 -v ./simple_api:/data --network=student-list-network student-list-img:v1

docker ps

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/e0de3cd0-34f2-422e-b511-b8275a8f3e9a)

3bis- run curl command to make sure that the API correctly responding

curl -u toto:python -X GET http://192.168.56.141:5000/pozos/api/v1.0/get_student_ages

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/d0f6dc9c-af2a-4eeb-baff-f127805f16cb)


4-create web container on the same network and using  persistent volume

Before running this container, we must modify the index.php file to enter the URL of the flask python API as well as the login and password for the connection

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/a7e9bc75-87ef-4b4e-a3fc-5051463e7f29)

Now we can create and execute the container

docker run --rm -d --name=student_list_web -p 8000:80 -v ./website:/var/www/html --network=student-list-network -e USERNAME=toto -e PASSWORD=python php:apache

docker ps 

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/d16c68eb-b5a9-44ff-a148-b8f870f84a51)


5-test web URL:

From the internet browser, enter the URL composed of the IP address(192.168.56.141) of the host and the port(8000)

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/d5cfe2d1-9dce-4e15-843f-88ca60896d72)


![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/64d5bfed-a61b-4bd2-aab8-43dfa650ed0d)





## Infrastructure As Code 


The ***docker-compose.yml*** file will deploy two services :

- website: the end-user interface with the following characteristics
   - image: php:apache
   - environment: you will provide the USERNAME and PASSWORD to enable the web app to access the API through authentication
   - volumes: to avoid php:apache image run with the default website, we will bind the website given by POZOS to use. You must have something like
`./website:/var/www/html`
   - depend on: you need to make sure that the API will start first before the website
   - port: do not forget to expose the port
- API: the image builded before should be used with the following specification
   - image: the name of the image builded previously
   - volumes: You will mount student_age.json file in /data/student_age.json
   - port: don't forget to expose the port
   - networks: don't forget to add specific network for your project

Delete the previous created container

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/d6b2a2d3-72d2-4e11-89b8-5fd4dcc0c99b)


Run docker-compose.yml

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/f6154137-5949-4db4-92b3-bcbd630f75ea)


Finally, reach website and click on the bouton "List Student"

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/5a9168de-8928-422f-9988-7b827f30b41b)


**If the list of the student appears, you are successfully dockerizing the POZOS application! Congratulation (make a screenshot)**

## Docker Registry 

Previously I already created a local registry on another machine(IP: 192.168.56.142:8090), so I will use it to push the student-list-img image used for this project.

This is the docker compose yml file for creating this registry:

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/22540fff-715e-45bd-99c5-9bd40bcc4fd7)

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/16b95ae1-ea9d-4d1b-ab59-8d81baf0bb66)

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/dc8d5519-eb4c-4592-b477-6658b6c1532c)

I set up an ssh connection between the remote registry and the machine where the student-list project runs.

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/6fa780b0-b77f-44e3-90e5-18579bc9dc12)

We now tag and send the image to the remote registry.

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/a91ee28f-d94c-4ee2-a913-f334894e90c4)

It can be seen on the remote registry portal.

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/6b3c9261-a776-4cac-aa27-8bd44e0a37e0)

docker-compose-registry.yml will be added to the project directory.

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/e794c3ea-75ac-4d3f-a48d-c00570668999)

![image](https://github.com/ravelonanosy/mini-projet-docker-02/assets/138290448/2bf587d8-ff95-4cb9-9df9-c1e68f38824d)


## Delivery 

Your delivery must be zip named firstname.zip (replace firstname by your own) that contain:

- A doc or PDF file with your screenshots and explanations.
- Configuration files used to realize the graded exercise (docker-compose.yml and Dockerfile).

Your delivery will be evaluated on:

- Explanations quality
- Screenshots quality (relevance, visibility)
- Presentation quality

Send your delivery at ***eazytrainingfr@gmail.com*** and we will provide you the link of the solution.

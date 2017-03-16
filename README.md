# Job Import

This project is a test using Docker with two services (described below), that are responsible to import a `job.txt` file to a PostgreSQL database and expose resources using a REST API, made with Rails 5 API.
The communication between the two services is made using RabbitMQ.
This project also runs on Docker, using `docker-compose`.

## Definition of services
The two services are:
- [Job API](https://github.com/skateonrails/job_api), that expose a REST api that should be used to see/create/active jobs;
- [Job Queue](https://github.com/skateonrails/job_queue), that is responsible to import the `jobs.txt` file to the Job API database sending messages to RabbitMQ that will be consumed by a [Sneakers](https://github.com/jondot/sneakers) worker on Job API.

## Constraints of this project
The constraints of each service is listed on it's own README file.

## How to run
Bellow are the following steps to run the project on your machine:

### Dependencies
You must have installed in your machine, at least, Git and Docker.

### Installation

- First, clone the repository :

  `git clone https://github.com/skateonrails/job_import`

- After that, get into `job_import` folder that was created and run the following commands:

  `git submodule update --init --recursive`

  This command is responsible to load the code for the two services into it's own folder.

- Now, start all services using:

  `docker-compose up`

### Setup
After a long wait, with previous commands you'll have all services (Job API, Job Queue, PostgreSQL and RabbitMQ) up and running.  
Now, it's time to load some dependencies.  
In another terminal window, at the same folder, you should use the following commands to setup database:

`docker-compose run job_api bash -c "rake db:create && rake db:migrate && rake db:seed"`

It will create the database structure needed by the Job API to run. It also create a `User` record, that we will use to access the protected API (see table).

To load the `jobs.txt` file into the api, you should use the command:

`docker-compose run job_queue ./start.sh`

This command will parse the `jobs.txt` file and send it's content to RabbitMQ, that will be consumed by Job API workers.

Congratulations, now you can access the API!

### Consuming the API
The API will be running on `http://localhost:3000`, and have the following endpoints:

| Name       | Method    | URL                  | Protected |
| ---        | ---       | ---                  | :--:      |
| List       | GET       | /jobs                | ✓         |
| Create     | POST      | /jobs                | ✓         |
| Activate   | POST      | /jobs/{:id}/activate | ✓         |
| Percentage | GET       | /category/{:id}      | ✘         |

You can access the API using the `curl` command tool, as the following:

- GET List jobs

  `curl -H "Authorization: Token token=Ab0CprY2MtCnFzHrbkXWHAtt" http://localhost:3000/jobs`

- POST Create jobs

  `curl -H "Content-Type: application/json" -H "Authorization: Token token=Ab0CprY2MtCnFzHrbkXWHAtt" -X POST -d '{"partner_id": 1, "title": "Created Now", "category_id": 11, "expires_at": "2017-10-31", "state": "active"}' http://localhost:3000/jobs`

- POST Activate jobs

  `curl -H "Authorization: Token token=Ab0CprY2MtCnFzHrbkXWHAtt" -X POST  http://localhost:3000/jobs/1/activate`

- GET Percentage by category

  `curl http://localhost:3000/category/1`


The `Ab0CprY2MtCnFzHrbkXWHAtt` is a token created with seeds from API, that "simulates" a user token. If you create another user (maybe using `docker-compose run job_api rails console`), you could use the `api_key` attribute from the newly created `User` to log into protected routes.

### Running tests
To run specs, with all up and running and databases created (setup's first command), you can run specs using the following commands:

- To run Job API specs:
  `docker-compose run job_api rspec`
- To run Job Queue specs:
  `docker-compose run job_queue rspec`

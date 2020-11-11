# Terraform CI/CD Using Concourse

Terraform CI/CD pipelines using Concourse CI

## Setup Concourse locally

Download the docker-compose file for Concourse

```bash
wget https://concourse-ci.org/docker-compose.yml
```

Modify the docker-compose file and adapt to your need.

```yml
version: '3'

services:
  concourse-db:
    image: postgres
    environment:
      POSTGRES_DB: concourse
      POSTGRES_PASSWORD: concourse_pass
      POSTGRES_USER: concourse_user
      PGDATA: /database

  concourse:
    # e.g I want to use Concourse 5.8.1
    image: concourse/concourse:5.8.1
    command: quickstart
    privileged: true
    depends_on: [concourse-db]
    # e.g my 8080 port is used by other container, so use 9090 instead
    ports: ["9090:8080"]
    environment:
      CONCOURSE_POSTGRES_HOST: concourse-db
      CONCOURSE_POSTGRES_USER: concourse_user
      CONCOURSE_POSTGRES_PASSWORD: concourse_pass
      CONCOURSE_POSTGRES_DATABASE: concourse
      CONCOURSE_EXTERNAL_URL: http://localhost:9090
      # username and password for logging
      CONCOURSE_ADD_LOCAL_USER: test:test
      CONCOURSE_MAIN_TEAM_LOCAL_USER: test
      CONCOURSE_WORKER_BAGGAGECLAIM_DRIVER: overlay
      CONCOURSE_CLIENT_SECRET: Y29uY291cnNlLXdlYgo=
      CONCOURSE_TSA_CLIENT_SECRET: Y29uY291cnNlLXdvcmtlcgo=
```

Run the container using `docker-compose`

```bash
docker-compose up -d
```

Login and create your Concourse target using `fly` CLI

```bash
fly -t demo login -c http://localhost:9090 -u test -p test
```

Check out the targets to make sure it is created

```bash
fly targets
```

Go to `http://localhost:9090` and login

For more CLIs, check out this [link](https://concourse-ci.org/fly.html)

## Prepare Terraform code

See `terraform` directory

```bash
├── dev
│   └── demo-cicd
│       └── main.tf
├── modules
│   └── demo-cicd
│       ├── main.tf
│       └── variables.tf
└── stage
    └── demo-cicd
        └── main.tf
```

## Create Concourse pipeline

See `.ci` directory

```bash
├── pipeline.yml
└── tasks
    ├── tf-deploy.yml
    ├── tf-format.yml
    └── tf-plan.yml
```

### Manage pipeline

You need to set pipeline to apply your YAML configuration change

```bash
# Set pipeline with pipeline name, pipeline config YAML file and vars
fly -t demo set-pipeline -p concourse-tf-cicd -c pipeline.yml -v github-access-token="xxxx" -v aws-access-key="xxx" -v aws-secret-key="xxx" -v slack-web-hook="xxx"
```

You can also destroy the pipeline when needed

```bash
# Destroy pipeline with pipeline name and pipeline config YAML file
fly -t demo destroy-pipeline -p concourse-tf-cicd -c pipeline.yml
```

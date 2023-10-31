## Gitlab CI

### Overview
- A tool provided by Gitlab to automate the lifecycle of an application.


### Usage
- The procedure is defined in `.gitlab-ci.yml` which resides in the root of your repository.
- Runners: These are agents or virtual machines that execute the jobs. They pick up jobs based on tags and other configurations. Runners can be specific to one project or can be shared among several projects. The runners are either managed or on-prem servers (pods/VMs).
- Configure GitLab runner:
  VM:
  ```
  sudo yum install gitlab-runner
  sudo gitlab-runner register
  sudo gitlab-runner start
  ```
  Container:
  ```
  docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
  docker exec -it gitlab-runner gitlab-runner register
  ```
  Pod:
  ```
  helm install gitlab-runner gitlab/gitlab-runner \
  --set gitlabUrl=YOUR_GITLAB_URL \
  --set runnerRegistrationToken=REGISTRATION_TOKEN \
  --namespace gitlab
  ```
- The jobs are running on a container created from the specified image, which allows us to provide the necessary dependencies for the operations of the job.

### Syntax
- Pipelines: A pipeline is the top-level structure that represents the results of one run for a project. It can contain multiple jobs and stages.
- Stages: A stage is a phase in the CI process, like build, test, and deploy. Jobs of the same stage run in parallel (if there are sufficient runners), and jobs of the next stage run after the jobs from the previous stage are completed successfully.
```
stages:
  - lint
  - test
  - build
  - deploy
```
- Jobs: A job is a set of instructions that the runner (a machine) has to execute. Jobs are defined in the .gitlab-ci.yml file in your repository.
```
# Lint the codebase
lint-frontend:
  stage: lint
  script:
    - cd frontend
    - npm install
    - npm run lint

lint-backend:
  stage: lint
  script:
    - cd backend
    - npm install
    - npm run lint

# Test the codebase
test-backend:
  stage: test
  image: node:latest
  services:
    - postgres:latest
  script:
    - cd backend
    - npm install
    - npm run test

# Build the codebase
build-frontend:
  stage: build
  script:
    - cd frontend
    - npm install
    - npm run build
  artifacts:
    paths:
      - frontend/build/

build-backend:
  stage: build
  script:
    - cd backend
    - npm install
    - npm run build
  artifacts:
    paths:
      - backend/dist/

# Deploy to Staging & Production
deploy-staging:
  stage: deploy
  environment:
    name: staging
  script:
    - echo "Deploying to Staging..."
    # Your deployment scripts to the staging environment

deploy-production:
  stage: deploy
  environment:
    name: production
  when: manual
  script:
    - echo "Deploying to Production..."
    # Your deployment scripts to the production environment
  only:
    - master
```
- Artifacts: After a job runs, it can produce artifacts (files). These artifacts can be passed between jobs and can be downloaded after a job is completed.
```
test:
  script: make test
  artifacts:
    paths:
      - test-reports/
    expire_in: 1 week
```
- Cache: Cache is used between runs to speed up time-consuming operations by reusing previously generated results or data, such as dependencies.
```
build:
  script: make build
  cache:
    paths:
    - .build/
```
- Variables: Variables are declared in the pipeline and can be used in the jobs.
```
variables:
  POSTGRES_DB: mydb
  POSTGRES_USER: user
  POSTGRES_PASSWORD: password
```
```
test:
  script:
    - export DB_CONNECTION=$DATABASE_URL
    - npm run test
```
- Environment: The environment represents a place where the application runs. Environments allow management and track deployments across different stages of the software's lifecycle. By segmenting into different environments, we can safely test and stage our application before it reaches production.
```
deploy_staging:
  stage: deploy
  script: ./deploy_to_staging.sh
  environment:
    name: staging
    url: https://staging.example.com
```
The deploy_staging job deploys to the staging environment, and GitLab will automatically link to https://staging.example.com when you view this environment in the GitLab UI.

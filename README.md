# VProfile Application
###
VProfile is a Java Spring MVC web application packaged as a WAR and deployed on Apache Tomcat. It includes user registration, login, and profile pages, with integrations configured for MySQL, Memcached, RabbitMQ, and Elasticsearch.

## Technology stack

- Java 21
- Maven
- Spring MVC and Spring Security
- JSP views
- Apache Tomcat 10
- MySQL, Memcached, RabbitMQ, and Elasticsearch integrations
- Docker and GitHub Actions for build and deployment automation

## Prerequisites

- JDK 21
- Maven 3.9 or later
- Docker, if building or running the container image
- Reachable MySQL, Memcached, RabbitMQ, and Elasticsearch services for an integrated deployment

## Build and test

Run the Maven verification lifecycle, including unit tests:

```sh
mvn clean verify
```

The WAR is generated at `target/vprofile-v2.war`.

Run only the controller tests with:

```sh
mvn -Dtest=UserControllerTest test
```

JUnit 4 tests are run through the JUnit Vintage engine; the project also uses JUnit 5 dependencies.

## Configuration

Runtime service settings are currently in [application.properties](src/main/resources/application.properties). Configure database URL and credentials, Memcached hosts, RabbitMQ address and credentials, and Elasticsearch host and port to match the deployment environment.

The checked-in configuration contains development defaults. Do not use those credentials in production. Prefer injecting credentials and environment-specific settings through a secret manager or deployment configuration rather than committing secrets.

Database initialization SQL is provided in [accountsdb.sql](src/main/resources/accountsdb.sql) and [db_backup.sql](src/main/resources/db_backup.sql). Review the SQL and configure the database before starting the application.

## Docker

The multistage image builds the WAR from this repository and runs it on Tomcat 10 with Java 21. Build from the repository root:

```sh
docker build -f Docker-files/app/multistage/Dockerfile -t vprofile:local .
```

Run the image, making sure the required backing services are reachable and configured for the container:

```sh
docker run --rm -p 8080:8080 vprofile:local
```

Open <http://localhost:8080/>. A local container by itself does not provide the external database, cache, message broker, or search service.

## Continuous integration and delivery

The workflow is defined in [.github/workflows/ci.yml](.github/workflows/ci.yml):

- Pull requests targeting `main` run Maven verification and Checkstyle, then scan using the configured self-hosted SonarQube server and wait for its quality gate.
- Pushes to `main` build the Docker image, push both the full commit SHA tag and `latest` to Amazon ECR, and update the app image and SHA tag in the Helm GitOps repository.
- Feature branch pushes do not trigger this workflow. The Docker and Helm update jobs are limited to pushes to `main`.

Configure these repository variables:

| Variable | Purpose |
| --- | --- |
| `SONAR_HOST_URL` | URL of the self-hosted SonarQube server |
| `AWS_REGION` | AWS region (defaults to `us-east-1` in the workflow) |
| `ECR_REPOSITORY` | ECR repository name (defaults to `vprofileappimg`) |
| `HELM_REPO_NAME` | Helm GitOps repository name (defaults to `vprofile-helm`) |

Configure these GitHub Actions secrets:

| Secret | Purpose |
| --- | --- |
| `SONAR_TOKEN` | Authentication token for SonarQube analysis and quality-gate checks |
| `AWS_ACCESS_KEY_ID` | AWS identity used to authenticate to ECR |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key used to authenticate to ECR |
| `HELM_REPO_USER` | GitHub username/owner for the Helm repository |
| `GITOPS_PAT` | Personal access token with permission to push to the Helm repository |

The workflow expects the GitOps repository to contain `helm/vprofile/values.yaml` with `app.image` and `app.tag` fields. SonarQube project settings are in [sonar-project.properties](sonar-project.properties).

## Repository layout

- `src/main/java/` — Spring MVC application code
- `src/main/resources/` — application configuration and database scripts
- `src/main/webapp/WEB-INF/views/` — JSP views and Spring XML configuration
- `src/test/java/` — unit and MVC tests
- `Docker-files/app/multistage/Dockerfile` — Maven build and Tomcat runtime image
- `.github/workflows/ci.yml` — pull-request CI and main-branch deployment workflow

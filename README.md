# Infrastructure Lab

A hands-on infrastructure and CI/CD project built to develop practical skills in Linux administration, Java application deployment, Apache Maven, Apache Tomcat, Git/GitHub, and Jenkins automation.

The project demonstrates an automated deployment pipeline with application verification, persistent deployment state, automatic rollback, fail-fast validation, and deployment safety controls.

## Architecture

```text
Developer
    |
    v
Git / GitHub
    |
    v
Jenkins Pipeline
    |
    +---- Load Deployment State
    |
    +---- Build with Maven
    |
    +---- Deploy WAR
    |
    v
Apache Tomcat
    |
    +---- Verify Application
    |
    +---- Success ----> Update Known-Good Version
    |
    +---- Failure ----> Automatic Rollback
```

## Technologies Used

- Ubuntu Linux
- Git
- GitHub
- Java / OpenJDK 21
- Apache Maven
- Apache Tomcat 10
- Jenkins
- Jenkins Declarative Pipeline
- Bash
- curl

## Application

The project contains a simple Java web application packaged as a WAR file using Maven.

Example artifact:

```text
infrastructure-lab-1.2.war
```

The WAR is deployed to:

```text
/var/lib/tomcat10/webapps/
```

Tomcat serves the application on port `8080`.

## Jenkins CI/CD Pipeline

The Jenkins pipeline is defined as code in:

```text
Jenkinsfile
```

The pipeline performs the following stages:

1. Environment validation
2. Load deployment state
3. Build
4. Deploy
5. Verify
6. Update known-good version
7. Rollback when required

## Environment Validation

The pipeline checks the Jenkins execution environment and displays information including:

```text
Jenkins user
Workspace
Java version
Maven version
```

This helps confirm that the build agent has the required tools before deployment.

## Persistent Deployment State

The last verified healthy application version is stored outside the Jenkins workspace:

```text
/var/lib/jenkins/deployment-state/infrastructure-lab-known-good
```

Example:

```text
1.2
```

Because this state is stored outside the Jenkins workspace, it remains available between pipeline executions.

## Fail-Fast State Validation

Before building or deploying an application, Jenkins validates the known-good deployment state.

The pipeline stops immediately if:

- the known-good state file does not exist
- the known-good state file is empty

This prevents Jenkins from performing a deployment when it does not have a valid rollback target.

## Build

The requested application version is supplied through the Jenkins parameter:

```text
APPLICATION_VERSION
```

Maven updates the project version and creates the WAR:

```bash
mvn versions:set -DnewVersion=<version>
mvn clean package
```

## Deployment

Jenkins copies the generated WAR into the Tomcat deployment directory.

Example:

```text
target/infrastructure-lab-1.2.war
        |
        v
/var/lib/tomcat10/webapps/
```

Tomcat automatically deploys the application.

## Deployment Verification

After deployment, Jenkins verifies the application using `curl`.

The pipeline retries the health check while Tomcat deploys the WAR.

A deployment is considered healthy only when the expected application version is returned.

Conceptually:

```text
Deploy
  |
  v
Verify application
  |
  +---- Healthy ----> Continue
  |
  +---- Unhealthy --> Retry
                         |
                         v
                   Verification fails
                         |
                         v
                      Rollback
```

## Automatic Rollback

If deployment verification fails, Jenkins sets a deployment failure flag and skips the known-good update.

The rollback stage:

1. Reads the previously loaded known-good version.
2. Removes the failed WAR.
3. Checks the known-good application.
4. Retries the health check if necessary.
5. Confirms that the previous application is healthy.

Example:

```text
Deploy 1.3
    |
    v
Verification FAILED
    |
    v
Remove failed 1.3 WAR
    |
    v
Verify known-good 1.2
    |
    v
ROLLBACK SUCCESSFUL
```

## Pipeline Results

The pipeline distinguishes between three important operational outcomes.

### SUCCESS

```text
Deployment successful
Application verification successful
Known-good version updated
```

### UNSTABLE

```text
Deployment failed
Rollback successful
Service recovered
```

An unstable result indicates that the requested deployment was unsuccessful even though the previous healthy application was recovered.

### FAILURE

```text
Build/deployment failure that cannot be safely recovered
```

## Concurrent Deployment Protection

The pipeline uses:

```groovy
disableConcurrentBuilds()
```

This prevents multiple executions of the same Jenkins pipeline from modifying the Tomcat deployment and persistent deployment state simultaneously.

## Failure Scenarios Tested

The project was tested using controlled failures rather than only successful deployments.

Tests included:

- Successful deployment of version 1.2
- Failed deployment of version 1.3
- HTTP 404 while Tomcat was deploying
- HTTP 500 caused by intentionally invalid JSP content
- Automatic rollback from failed 1.3 to healthy 1.2
- Missing known-good state file
- Empty known-good state file
- Successful rollback reported as Jenkins `UNSTABLE`

These tests verified both the normal deployment path and multiple failure paths.

## Key Lessons

This project provided hands-on experience with:

- Linux service administration
- Java runtime environments
- Maven build lifecycle
- WAR packaging
- Tomcat application deployment
- Git version control
- GitHub repositories
- Jenkins Pipeline as Code
- CI/CD pipeline design
- Health checks
- Retry logic
- Persistent deployment state
- Fail-fast validation
- Automated rollback
- Failure testing
- Deployment concurrency protection

## Future Development

The next stage of this lab environment will expand toward Site Reliability Engineering and Platform Engineering concepts, including:

- Infrastructure as Code with Terraform
- Configuration management with Ansible
- Docker containers
- Kubernetes
- Cloud infrastructure
- Monitoring and observability
- Reliability engineering practices
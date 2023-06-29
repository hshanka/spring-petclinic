# Hari's Spring PetClinic Application
This project is a fork of the Spring Petclinic application provided as part of the exercise. This readme describes the changes made to this project in order to meet the exercise requirements. Specifically, the following tasks were accomplished:

a) Forked Spring-Petclinic project from https://github.com/spring-projects/spring-petclinic

b) Added a Dockerfile with steps to build the docker image

c) Modified pom.xml to point to JCenter. Ensured all dependencies are resolving from JCenter

d) Added a Jenkinsfile to the project to compile the code, run the tests and package the application into a docker image

e) Added steps to the Jenkinsfile to publish the docker image to Artifactory (bonus item)

f) Added steps to the Jenkinsfile to run vulnerability scans using JFrog Xray (bonus item)



## Detailed description of steps executed to complete the exercise

Once the application was forked, I cloned the repo locally to build and test the project. I executed. the following commands. todo so:

```
git clone https://github.com/spring-projects/spring-petclinic.git

cd spring-petclinic

./mvnw package

java -jar target/*.jar
```

I verified that the application was available at http://localhost:8080/. 

## Updating repos to point to JCenter:
I made the following changes to pom.xml:

**Removed:**
```
  <repositories>
    <repository>
      <id>spring-snapshots</id>
      <name>Spring Snapshots</name>
      <url>https://repo.spring.io/snapshot</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repository>
    <repository>
      <id>spring-milestones</id>
      <name>Spring Milestones</name>
      <url>https://repo.spring.io/milestone</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

   <pluginRepositories>
    <pluginRepository>
      <id>spring-snapshots</id>
      <name>Spring Snapshots</name>
      <url>https://repo.spring.io/snapshot</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </pluginRepository>
    <pluginRepository>
      <id>spring-milestones</id>
      <name>Spring Milestones</name>
      <url>https://repo.spring.io/milestone</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </pluginRepository>
  </pluginRepositories>

```

**Added:**
```
<repositories>
   <repository>
        <id>jcenter</id>
        <name>jcenter</name>
        <url>https://jcenter.bintray.com</url>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>spring-snapshots</id>
      <name>Spring Snapshots</name>
      <url>https://jcenter.bintray.com</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </pluginRepository>
    <pluginRepository>
      <id>spring-milestones</id>
      <name>Spring Milestones</name>
       <url>https://jcenter.bintray.com</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </pluginRepository>
  </pluginRepositories>

```

## Containerizing the Spring-Petclinic application

My next step was to continerize the application. In order to do so, I added a Dockerfile to the prject with the contents below:

```
# syntax=docker/dockerfile:1

FROM eclipse-temurin:17-jdk-jammy

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:resolve

COPY src ./src

CMD ["./mvnw", "spring-boot:run"]

```
I also added a .dockerignore to exclude the target/ directory from being included during docker image creation.

I built the docker image using the following command in the project's root directory:

```
docker build --tag spring-petclinic-hs .

```

I ran the image as a container locally using the command:

```
docker run --publish 8080:8080 spring-petclinic-hs
```

During startup, I verified that all dependencies were resolving from JCenter:

<img width="1155" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/32ac7e1e-02d7-4b94-99d8-e45fe1bbc5bc">


## Automation with Jenkins

### Installing Jenkins

Next, I installed a local Jenkins instance using homebrew:

```
 brew install jenkins-lts

```
I started the Jenkins server using:
```
brew services start jenkins-lts

```

I then installed the following plugins into my Jenkins server:
a) JFrog 
b) Docker Pipeilne
c) Github 

I configured Maven, Git and Docker and JFrog CLI within Jenkins Tools -> Congigurations.


### Authoring the Jenkinsfile
I created a Jenkinsfile with the following contents:
```
pipeline {
   
    agent any
    
    tools {
        maven 'maven'
        jfrog 'jfrog-cli'
    } 
    
    environment {
		DOCKER_IMAGE_NAME = "hshanka.jfrog.io/frog-docker/spring-petclinic-hs:$BUILD_NUMBER"
	}
    
    stages {
       stage('Cloning Git') {
         steps {
             checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '1915d014-8308-45ab-bec7-bb5c7bb2a0e1', url: 'https://github.com/hshanka/spring-petclinic.git']])
            }
        }
        
    stage('Compile Code, Run Tests, Build App') {
            steps {
                sh 'mvn clean package'
            }
        } 
    stage('Build Docker Image') {
            steps {
                 script {
                    def dockerImage = docker.build("$DOCKER_IMAGE_NAME", '-f Dockerfile .')
                }
            }
        } 
        
    stage('Scan and push image') {
			steps {
			
					// Scan Docker image for vulnerabilities
					jf 'docker scan $DOCKER_IMAGE_NAME'

					// Push image to Artifactory
					jf 'docker push $DOCKER_IMAGE_NAME'
				
			}
		}

    }
}
```

As seen above, the script checks out the code from my repo, compiles, runs the tests and attempts to build the package. If these steps succeed, the script builds a docker image using the Dockerfile I created and placed at the root of the project.

## Scanning for security vulnerabilities (Bonus Task)
After the docker image is built, I scan for vulnerabilities using JFrog Xray within the Jenkinsfile using,

```
// Scan Docker image for vulnerabilities
jf 'docker scan $DOCKER_IMAGE_NAME'
```
The scan report is output on my Jenkins job conole during builds:

<img width="1107" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/7d77a72c-812f-4a14-8183-f6b56675024b">

The Xray scan results are also available within the Xray section within my JFrog trial account.

<img width="1411" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/bc930b95-770b-4ab5-8994-39f4c131328a">

<img width="1463" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/e8b792da-de4f-4086-8181-f9ef31e51126">

<img width="1462" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/bf76bb02-f43e-4a02-b561-9ce9cefacb41">

<img width="1458" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/45bee72d-cf08-41a4-81d4-777664af06a2">



## Publishing Docker image to JFrog Artifactory (Bonus Task)

Once Xray scans complete, I publish the docker image to Artifactory using,
```
// Push image to Artifactory
jf 'docker push $DOCKER_IMAGE_NAME'
```
I use the build number as the automatic version number for each subsequent image build and publish. 
Builds are visible in Artifactory within my JFrog account:

<img width="1465" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/99e10359-1947-453c-b80c-e983b77992ca">


# Deliverables

### 1. Jenkinsfile
This is available in the root of this repo at: https://github.com/hshanka/spring-petclinic/blob/main/Jenkinsfile

### 2. Dockerfile
This is available in the root of this repo at: https://github.com/hshanka/spring-petclinic/blob/main/Dockerfile


## Compiling the CSS

There is a `petclinic.css` in `src/main/resources/static/resources/css`. It was generated from the `petclinic.scss` source, combined with the [Bootstrap](https://getbootstrap.com/) library. If you make changes to the `scss`, or upgrade Bootstrap, you will need to re-compile the CSS resources using the Maven profile "css", i.e. `./mvnw package -P css`. There is no build profile for Gradle to compile the CSS.

## Working with Petclinic in your IDE

### Prerequisites
The following items should be installed in your system:
* Java 17 or newer (full JDK, not a JRE).
* [git command line tool](https://help.github.com/articles/set-up-git)
* Your preferred IDE 
  * Eclipse with the m2e plugin. Note: when m2e is available, there is an m2 icon in `Help -> About` dialog. If m2e is
  not there, follow the install process [here](https://www.eclipse.org/m2e/)
  * [Spring Tools Suite](https://spring.io/tools) (STS)
  * [IntelliJ IDEA](https://www.jetbrains.com/idea/)
  * [VS Code](https://code.visualstudio.com)

### Steps:

1) On the command line run:
    ```
    git clone https://github.com/spring-projects/spring-petclinic.git
    ```
2) Inside Eclipse or STS:
    ```
    File -> Import -> Maven -> Existing Maven project
    ```

    Then either build on the command line `./mvnw generate-resources` or use the Eclipse launcher (right click on project and `Run As -> Maven install`) to generate the css. Run the application main method by right-clicking on it and choosing `Run As -> Java Application`.

3) Inside IntelliJ IDEA
    In the main menu, choose `File -> Open` and select the Petclinic [pom.xml](pom.xml). Click on the `Open` button.

    CSS files are generated from the Maven build. You can build them on the command line `./mvnw generate-resources` or right-click on the `spring-petclinic` project then `Maven -> Generates sources and Update Folders`.

    A run configuration named `PetClinicApplication` should have been created for you if you're using a recent Ultimate version. Otherwise, run the application by right-clicking on the `PetClinicApplication` main class and choosing `Run 'PetClinicApplication'`.

4) Navigate to Petclinic

    Visit [http://localhost:8080](http://localhost:8080) in your browser.


## Looking for something in particular?

|Spring Boot Configuration | Class or Java property files  |
|--------------------------|---|
|The Main Class | [PetClinicApplication](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/java/org/springframework/samples/petclinic/PetClinicApplication.java) |
|Properties Files | [application.properties](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/resources) |
|Caching | [CacheConfiguration](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/java/org/springframework/samples/petclinic/system/CacheConfiguration.java) |

## Interesting Spring Petclinic branches and forks

The Spring Petclinic "main" branch in the [spring-projects](https://github.com/spring-projects/spring-petclinic)
GitHub org is the "canonical" implementation based on Spring Boot and Thymeleaf. There are
[quite a few forks](https://spring-petclinic.github.io/docs/forks.html) in the GitHub org
[spring-petclinic](https://github.com/spring-petclinic). If you are interested in using a different technology stack to implement the Pet Clinic, please join the community there.


## Interaction with other open source projects

One of the best parts about working on the Spring Petclinic application is that we have the opportunity to work in direct contact with many Open Source projects. We found bugs/suggested improvements on various topics such as Spring, Spring Data, Bean Validation and even Eclipse! In many cases, they've been fixed/implemented in just a few days.
Here is a list of them:

| Name | Issue |
|------|-------|
| Spring JDBC: simplify usage of NamedParameterJdbcTemplate | [SPR-10256](https://jira.springsource.org/browse/SPR-10256) and [SPR-10257](https://jira.springsource.org/browse/SPR-10257) |
| Bean Validation / Hibernate Validator: simplify Maven dependencies and backward compatibility |[HV-790](https://hibernate.atlassian.net/browse/HV-790) and [HV-792](https://hibernate.atlassian.net/browse/HV-792) |
| Spring Data: provide more flexibility when working with JPQL queries | [DATAJPA-292](https://jira.springsource.org/browse/DATAJPA-292) |


# Contributing

The [issue tracker](https://github.com/spring-projects/spring-petclinic/issues) is the preferred channel for bug reports, features requests and submitting pull requests.

For pull requests, editor preferences are available in the [editor config](.editorconfig) for easy use in common text editors. Read more and download plugins at <https://editorconfig.org>. If you have not previously done so, please fill out and submit the [Contributor License Agreement](https://cla.pivotal.io/sign/spring).

# License

The Spring PetClinic sample application is released under version 2.0 of the [Apache License](https://www.apache.org/licenses/LICENSE-2.0).

[spring-petclinic]: https://github.com/spring-projects/spring-petclinic
[spring-framework-petclinic]: https://github.com/spring-petclinic/spring-framework-petclinic
[spring-petclinic-angularjs]: https://github.com/spring-petclinic/spring-petclinic-angularjs 
[javaconfig branch]: https://github.com/spring-petclinic/spring-framework-petclinic/tree/javaconfig
[spring-petclinic-angular]: https://github.com/spring-petclinic/spring-petclinic-angular
[spring-petclinic-microservices]: https://github.com/spring-petclinic/spring-petclinic-microservices
[spring-petclinic-reactjs]: https://github.com/spring-petclinic/spring-petclinic-reactjs
[spring-petclinic-graphql]: https://github.com/spring-petclinic/spring-petclinic-graphql
[spring-petclinic-kotlin]: https://github.com/spring-petclinic/spring-petclinic-kotlin
[spring-petclinic-rest]: https://github.com/spring-petclinic/spring-petclinic-rest

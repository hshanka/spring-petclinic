# Hari's Spring PetClinic Application
This project is a fork of the Spring Petclinic application provided as part of the exercise. This readme describes the changes made to this project in order to meet the exercise requirements. Also included below are the deliverables for this exercise.

The following tasks were accomplished:

a) Forked Spring-Petclinic project from https://github.com/spring-projects/spring-petclinic

b) Added a Dockerfile with steps to build the docker image

c) Modified pom.xml to point to JCenter. Ensured all dependencies are resolving from JCenter

d) Added a Jenkinsfile to the project to compile the code, run the tests and package the application into a docker image

e) Added steps to the Jenkinsfile to publish the docker image to Artifactory (bonus item)

f) Added steps to the Jenkinsfile to run vulnerability scans using JFrog Xray, as well as static code analysis using SonarQube (bonus item)


# Deliverables

### 1. Jenkinsfile
This is available in the root of this repo at: https://github.com/hshanka/spring-petclinic/blob/main/Jenkinsfile

### 2. Dockerfile
This is available in the root of this repo at: https://github.com/hshanka/spring-petclinic/blob/main/Dockerfile


### 3. Runnable Docker image:

#### Option A: Downloading From Artifactory
I added Marcus, Danh and Niecolle to my JFrog instance so you are able to pull the image from my Artifactory repo.

Pull the docker image from Artifactory using:
```
docker pull hshanka.jfrog.io/frog-docker/spring-petclinic-hs:9  
```
#### Option B: Download From Google Drive
The downloadable tar is also available on my [Google Drive](https://drive.google.com/drive/folders/1qJOGdtAxKLZEx7xKAjEkht9ceJTE_5h-?usp=sharing)

After downloading the image, execute the following command to load the docker image from the tar into your local images list.
```
docker load < hs-springpetclinic.tar
```

#### Option C: Download Via Greenhouse
The image has also been uploaded to the link provided via Greenhouse.

After downloading the image, execute the following command to load the docker image from the tar into your local images list.
```
docker load < hs-springpetclinic.tar
```

#### After loading the image, proceed with the steps below to run the container.

Verify that the image is available in your list of local docker images:

```
docker images
```

You should see ```hshanka.jfrog.io/frog-docker/spring-petclinic-hs:9``` in the list of your local images as seen below:
<img width="889" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/2604cf63-edef-4b13-95f7-d3363c0f1cd2">


Now run the container:

```
docker run --publish 8086:8080 hshanka.jfrog.io/frog-docker/spring-petclinic-hs:9
```
(Change 8086 to an available port on your local machine if that port is taken.)

Access http://localhost:8086/ to verify that the application came up successfully. 

<img width="1468" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/f94b080f-002f-4748-a518-8e04fbb8fbfe">

### 4. Slides covering the implementation and additional recommendations for security within the pipeline
Please see [here.](https://docs.google.com/presentation/d/1wsDOpUZpE5hqVn8xseXkofG8vAPbUrTmvsnBY3BLgs4/edit?usp=sharing)


# Description of steps executed to complete the exercise

## 1. Fork and clone project
Once the application was forked, I cloned the repo locally to build and test the project. I executed. the following commands. todo so:

```
git clone https://github.com/spring-projects/spring-petclinic.git

cd spring-petclinic

./mvnw package

java -jar target/*.jar
```

I verified that the application was available at http://localhost:8080/. 

## 2. Updating repos to point to JCenter:
I made the following changes to pom.xml:

### Removed:
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

### Added:
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

## 3. Containerizing the Spring-Petclinic application

My next step was to containerize the application. In order to do so, I added a Dockerfile to the project with the contents below:

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


## 4. Automation with Jenkins

### 4a. Installing Jenkins

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
b) Docker Pipeline
c) Github 

I configured Maven, Git and Docker and JFrog CLI within Jenkins Tools -> Configurations.


### 4b. Authoring the Jenkinsfile
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
        
        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv('') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=spring-petclinic -Dsonar.projectName='spring-petclinic'"
                }
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
                        docker.build("$DOCKER_IMAGE_NAME", '-f Dockerfile .')
                    }
                }
        } 
            
        stage('Scan and push image') {
                steps {
                
                        // Scan Docker image for vulnerabilities using JFrog Xray
                        jf 'docker scan $DOCKER_IMAGE_NAME'

                        // Push final docker image to Artifactory
                        jf 'docker push $DOCKER_IMAGE_NAME'
                    
                }
        }

    }
}
```
As seen above, the script checks out the code from my repo, compiles, runs the tests and attempts to build the package. If these steps succeed, the script builds a docker image using the Dockerfile I created and placed at the root of the project.

### 4c. Configuring Jenkins And Running The Pipeline
I pushed the updates into Github, and configured my Jenkins pipeline to point to this github project.
<img width="1211" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/368f5e7c-716d-4505-9aaa-eff63999fb88">

And ran the build manually.
<img width="1059" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/087e245c-d0e0-4333-b41e-82624af24dac">


## 5. Adding Security to Jenkins Pipeline (Bonus Task)

#### 5a. Scanning for security vulnerabilities 
After the docker image is built, I scan for vulnerabilities using JFrog Xray within the Jenkinsfile.

```
// Scan Docker image for vulnerabilities
jf 'docker scan $DOCKER_IMAGE_NAME'
```
The Xray scan report is output on my Jenkins job console during builds:

<img width="1107" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/7d77a72c-812f-4a14-8183-f6b56675024b">

The scan results are also available within the Xray section within my JFrog account.

<img width="1411" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/bc930b95-770b-4ab5-8994-39f4c131328a">


<img width="1463" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/e8b792da-de4f-4086-8181-f9ef31e51126">


<img width="1462" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/bf76bb02-f43e-4a02-b561-9ce9cefacb41">


<img width="1458" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/45bee72d-cf08-41a4-81d4-777664af06a2">

### 5b. Static Code Analysis Using SonarQube
I installed the SonarQube scanner plugin into my Jenkins environment.
![image](https://github.com/hshanka/spring-petclinic/assets/6666290/8ef33741-f855-46db-ab1c-f3ba1a4bebac)

Incorporated the scanner into my Jenkins pipeline. Static Code analysis results were published to my local SonarQube server.

<img width="1463" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/f10ea967-a011-4aa2-b4c2-02aa7b7e971b">

<img width="1459" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/18905a1c-5dbd-4587-86a3-7dd59b2a3ecd">


### 5c. Additional approaches to adding security to Jenkins pipeline
Please see [this deck](https://docs.google.com/presentation/d/1wsDOpUZpE5hqVn8xseXkofG8vAPbUrTmvsnBY3BLgs4/edit?usp=sharing) for additional approaches that I would implement to further secure this pipeline.


## 6. Publishing Docker image to JFrog Artifactory (Bonus Task)

Once Xray scans complete, I published the docker image to Artifactory.
```
// Push image to Artifactory
jf 'docker push $DOCKER_IMAGE_NAME'
```
I used the build number for automatic version update for each subsequent docker image build and publish. 
Builds are visible in Artifactory within my JFrog account:

<img width="1465" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/99e10359-1947-453c-b80c-e983b77992ca">


# Issues encountered and resolutions
### 1. Jenkins server not finding my local docker despite configuring Manage Plugins --> Tools.
I resolved this by editing 
```
/opt/homebrew/Cellar/jenkins-lts/2.401.1/homebrew.mxcl.jenkins-lts.plist
```
to add,
```
<dict>
<key>PATH</key>
<string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/Docker.app/Contents/Resources/bin/:/Users/hyadav23/Library/Group\ Containers/group.com.docker/Applications/Docker.app/Contents/Resources/bin</string>
</dict>
```
### 2. Port conflicts while running tests for MySQL and PostgreSQL
Shut down locally running databases, killed hanging processes.

### 3. Running into runtime error while using Frogbot
While running the following stage,
```
 stage('Scan and Fix Repos') {
  steps {
      sh "./frogbot scan-and-fix-repos"
      // For Windows runner:
      // powershell """.\frogbot.exe scan-and-fix-repos"""
  }    
```
I experienced the following runtime error:
<img width="1079" alt="image" src="https://github.com/hshanka/spring-petclinic/assets/6666290/2ee651b0-a1e9-41a2-90f6-794e818d174b">

I ended up removing frogbot from my Jenkinsfile since I was directly running the vulnerability scan using the jf commandline and Xray.


# Real-World Enhancement Ideas

### 1. Kick of Jenkins job on code commit
Instead of kicking off builds manually, we would want to trigger the Jenkins pipeline on each code commit.

### 2. Email build results to DevSecOps teams
This provides immediate visibility to important items like vulnerability scan results so the team can prioritize and expedite remediation.

### 3. Push results into an SRE observability dashboard
If DevOps/SRE teams use an observability solution like Grafana, Chronosphere or others, push the build logs and metrics into these solutions for immediate visibility and actionability.


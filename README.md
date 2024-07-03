### DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL

#### Architecture Diagram for three tier Application Deployment
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/ca1f541b-b2b4-470f-8780-eb49a51b0066)
Using the Terraform Script present in this repository create the end-to-end Infrastructure. 
<br><br/>
For RabbitMQ installation do as shown in screenshot below.
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/50369e84-61c1-4a58-9ae6-69dcf2f245d4)
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/fac5dbb8-a646-44d0-9f5d-556a514df5e8)
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/47582408-de11-4fce-8f76-e01c0164b7f6)
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/3f921f14-c7ce-425b-9c64-b5dc50448c38)
<br><br/>
Run the below command to create RDS MySQL database entry. To do so first of all create a file db.sql using the content as shown in the screenshot below.
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/9c2d1d7c-ae74-4fcc-b262-1b5b9f552d77)
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/318e2e65-058d-402c-a745-1253f02ac566)
<br><br/>
Establish the passwordless authentication between Jenkins Slave Node and Tomcat Server as shown in the screenshot below.
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/fb482f50-8b0a-40d3-bb6b-3abaab517eaa)
```
pipeline{
    agent{
        node{
            label "Slave-1"
            customWorkspace "/home/jenkins/3tierapp/"
        }
    }
    environment {
        JAVA_HOME="/usr/lib/jvm/java-17-amazon-corretto.x86_64"
        PATH="$PATH:$JAVA_HOME/bin:/opt/apache-maven/bin:/opt/node-v16.0.0/bin:/usr/local/bin"
    }
    parameters { 
        string(name: 'COMMIT_ID', defaultValue: '', description: 'Provide the Commit ID') 
    }
    stages{
        stage("Clone-code"){
            steps{
                cleanWs()
                checkout scmGit(branches: [[name: '${COMMIT_ID}']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-cred', url: 'https://github.com/singhritesh85/Three-tier-WebApplication.git']])
            }
        }
        stage("SonarQubeAnalysis-and-Build"){
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh 'mvn clean package sonar:sonar'
                }
            }
        }
        //stage("Quality Gate") {
        //    steps {
        //        timeout(time: 1, unit: 'HOURS') {
                    //waitForQualityGate abortPipeline: true
        //            waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
        //        }
        //    }
        //}
        stage("Nexus-Artifact Upload"){
            steps{
                script{
                    def mavenPom = readMavenPom file: 'pom.xml'
                    def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "maven-snapshot" : "maven-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.visualpathit', nexusUrl: 'nexus.singhritesh85.com', nexusVersion: 'nexus3', protocol: 'https', repository: "${nexusRepoName}", version: "${mavenPom.version}"
                }    
            }
        }
        stage("Deployment"){
            steps{
                      sh 'scp -rv target/vprofile-v2.war tomcat-admin@10.10.4.225:/opt/apache-tomcat/webapps/ROOT.war'
            }
        }
    }
}
```
Before running the Jenkins Job change the **application.properties** file present at the path Three-tier-WebApplication/src/main/resources in the Repository https://github.com/kamalmohan217/Three-tier-WebApplication.git as shown in the screenshot below.
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/2b2ae78e-20cb-4c1e-9123-11713a19c41c)
<br><br/>
The Screenshot for SonarQube Analysis and Nexus Artifacts Upload is as shown below.
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/bc6efcfb-9518-4855-8c3c-4783ee3611df)
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/2f14fe20-56c0-4821-bd3c-313048b31ad2)
<br><br/>
Do the entry in Route53 as shown in the screenshot below.
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/9d7ff7ca-8cb3-4ae9-aeee-29519c2b0c1b)
<br><br/>
Finally screenshot shown below Access of the Application.
<br><br/>
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/11e0c5a1-e065-479b-8735-c78f532b5539)
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/773039a8-5c09-4611-a000-f342f8946538)
![image](https://github.com/kamalmohan217/DevOps-Project-3tier-Application-Deployment-Tomcat-RabbitMQ-Memcache-MySQL/assets/128888356/b409394b-67a2-41be-84e4-3d4a5f17363c)

<br><br/>
<br><br/>
```
Source Code:-  https://github.com/kamalmohan217/Three-tier-WebApplication.git
```
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
```
Reference:-  https://github.com/logicopslab/vprofile-project.git
```

# Automating-Deployment-of-an-E-commerce-website


 As a DevOps Engineer, I am tasked with designing and implementing a CI/CD pipeline using Jenkins to automate the deployment of a web application for a technology consulting firm. The primary goals are to achieve continuous integration, continuous deployment, scalability, and reliability.


 # STEP 1: JENKINS SERVER SET UP

 * Install jenkins on the dedicated server:
 ![]()
 
 using the folllowing set of commands to install jenkins on the server
 update package repositories and install JDK
Using the command 
    
    sudo apt update 
    sudo apt install default-jdk-headless
![1](./img/1.png)

* using the command below to install the jenkins

    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt-get install jenkins

 ![2](./img/1a.png)  

* check if jenkins was installed and is up and running
 
    sudo systemctl status jenkins
![3](./img/1b.png)


* create an EC2 instance(jenkins) on AWS and set the inbound rule to 8080
![4](./img/1c.png)
![4](./img/1d.png)
![4](./img/1e.png)

* input the jenkins instance ip address on the web browser and set up a jenkins account
![5](./img/1g.png)
note check the lib to locate your Admin Passowrd using the code below:
      
      sudo cat /var/lib/jenkins/secrets/initialAdminPassword

 afterwhich install pluging
![5](./img/1h.png)

Step 6 : Log in to jenkins
![5](./img/1f.png)


# STEP 2: SOURCE CODE MANAGEMENT REPOSITORY INTEGRATION

* Connect jenkins with the source code management (GitHub)

     i. from the jenkins dashboard menu click on new item
![2](./img/2.png)

      ii. create a Github repository "jenkins-scm" with a README.md file
 ![2](./img/2a.png)  
       
      iii. connect jenkins to jenkins-scm repository by pasting the repository url "https://github.com/Emmyace-hub/jenkins-scm.git" in the area below using the 'main' branch
![2](./img/2b.png)
       
       and add my jenkins username and password to the credentials
![2](./img/2c.png)
       
 * configure webhooks for automatic triggering of jenkins builds      
 configure the build trigger on jenkins in order to be able to run a new build anytime a change is made on the github repository
        
        i. click on configure and edit the configuration
 ![2](./img/2d.png) 

        ii. under 'trigger'select "Github hook trigger for GITScm polling"
 ![2](./img/2e.png)     

        iii. create a githubwebhook using the jenkins ip address'https://975b-86-131-115-109.ngrok-free.app' and port '8080'        
![2](./img/2f.png)
![2](./img/2g.png)
       
       iv. commit a message "2nd update" on my github repository README file a new build was launched automaitically by webhook and clicking on console of the new build #7 to see exactly where changes were made to verify
![2](./img/2h.png)
![2](./img/2i.png)       


# STEP 3: JENKINS FREESTYLE JOB FOR BUILDING AND RUNNING TEST
  
  * Add Build Steps
      
      i. Scroll to **"Build"** section and Click **"Add build step"** and select **"Execute shell"**
![3](./img/3.png)
    
      ii. Enter build and test command into the exceute shell box and save:

         #!/bin/bash
         echo "Building the project..."
         # Add your build commands here, e.g.:
         npm install

         echo "Running tests..."
         # Add your test commands here, e.g.:
         npm test
        
![3](./img/3a.png)

       
iii. create a package.json file on github repository contain the script below


           {
           "scripts": {
           "test": "echo \"Running tests...\" && exit 0"
             }
            }
![3](./img/3b.png)

iv. click on build now and check the console output to see if the job was built successfully
![3](./img/3c.png)



# STEP 4: DEVELOP A JENKINS PIPELINE FOR WEB APPLICATION
 
 *  **Create a New Pipeline Job**
   i. From the Jenkins dashboard, click **"New Item"**.
   ii. Enter a name  `webapp-pipeline`, select **"Pipeline"**, and click **OK**.
   ![4](./img/4.png)

   **Configure Pipeline Script**
   - In the job configuration, scroll to the **Pipeline** section.
   - Select **"Pipeline script"** and enter the script below into the pipeline:
     
         pipeline {
            agent any
         stages {
            stage('Checkout') {
              steps {
                // Clone your repository
                git branch: 'main', url: 'https://github.com/Emmyace-hub/jenkins-scm.git'
             }
          }
            stage('Install Dependencies') {
              steps {
                sh 'npm install'
              }
          }
            stage('Run Tests') {
               steps {
                sh 'npm test'
              }
          }
           stage('Build') {
             steps {
                // Add your build commands here
                sh 'echo "Building the application..."'
              }
          }
          stage('Deploy') {
            steps {
                // Add your deployment commands here
                sh 'echo "Deploying the application..."'
               }
             }
         }
          post {
           success {
            echo 'Pipeline completed successfully!'
         }
           failure {
            echo 'Pipeline failed.'
           }
          }
         }
![4](./img/4a.png)
verify the pipeline ran successfully by clicking on Build now and checking the console output
![4](./img/4b.png)

# STEP 5: DOCKER IMAGE CREATION AND RESISSTRY PUSH

i. **Add Dockerfile to Your Repository**
![5](./img/5.png)

ii.update the pipeline script to include the docker credentials
 

    pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // Jenkins Credentials ID
        DOCKER_IMAGE = 'emmyace7/webapp'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Emmyace-hub/jenkins-scm.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Building the application..."'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} ."
                    sh 'echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin'
                    sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                }
            }
        }
    }

    post {
        success {
            echo 'pipeline completed successfully!'
        }
        failure {
            echo 'pipeline failed.'
        }
    }
}

iii. **Configure Jenkins Credentials**
   - In Jenkins, go to **Manage Jenkins > Credentials**.
   - Add your Docker Hub username and password as a "Username with password" credential.
   - Use the ID `dockerhub-credentials` in your pipeline script.
![5](./img/5b.png)
![5](./img/5c.png)
![5](./img/5a.png)

iii. **Verify Docker Installation**
   - Make sure Docker is installed and the Jenkins user has permission to run Docker commands on your Jenkins server.
   
   docker --version
   sudo usermod -aG docker jenkins
   sudo systemctl restart docker
   sudo systemctl restart jenkins

![5](./img/5d.png)

iv. **Run the Pipeline**
   - Click **Build Now** in Jenkins.
   - Check the console output to verify the Docker image is built and pushed to Docker Hub.
![5](./img/5e.png)


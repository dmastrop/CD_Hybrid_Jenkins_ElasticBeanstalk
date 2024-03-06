pipeline {
    agent any
    //run directly on Jenkins server without any agent for now
    tools {
        maven "MAVEN3_9"
        // there are 2 maven versions in tools. One installed via ssh terminal is MAVEN3_6
        // MAVEN3_9 is auto installed from the Java Tools configuration
        jdk "OracleJDK11"
        // JDK8 is deprecated. Do not use it. 
    }

    // the environment block is used to specify the variable values that are referenced in the pom.xml and settings.xml
    // the pom.xml will be used by maven to build from the source. The settings will define the Nexus repo URL that the
    // pom.xml needs to access the central reposistory for the dependencies.....
    environment {
        // these values were defined during Nexus server setup
        // note that underscore for variable names instead of dash. Dash is illegal character for Jenkinsfile.
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        //NEXUS_PASSWORD = 'admin'
        //NEXUS_USER = "admin"
        //NEXUS_PASSWORD = "admin"
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        // note that NEXUSIP is the private IP of the Nexus EC2 instance. This does not change even after reboots,
        // unlike public IP.  Jenkins server and Nexus server are in the same AWS VPC and in same subnet
        NEXUSIP = '172.31.28.191'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        // the NEXUS_LOGIN was defined in the Jenkins credentials settings. This will be used later.
        NEXUS_LOGIN = 'nexuslogin'

    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
                // -DskipTests so it will not run any tests. It will run mvn install.  The settings.xml will be
                // passed to it as parameters.  Settings.xml is in this same workspace 
                // We want Jenkins to download dependencies from Nexus. That is specified in the pom.xml file in 
                // this workspace.  The Nexus repo is specified at the  end of the pom.xml
            }

            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                    //archiveArtifacts is a pliugin.  This states to archive everyting that ends in .war.
                    //  our .war file is in the workspace for the particular jenkins pipeline
                    // in the terminal is is under /var/lib/jenkins/workspace/vprofile-hkhcoder-ci-pipeline2/target
                }
            }
        }

        stage('Test'){
            steps {
                //sh 'mvn test'
                sh 'mvn -s settings.xml test'
                // this will generate a test. Later on we will configure this to place it on Sonarqube
                // note if do not add -s settings.xml then dependencies will be downloaded from maven instead
                // of Nexus repo.
            }
        }    

        stage('Checkstyle Analysis'){
            steps {
                //sh 'mvn checkstyle:checkstyple'
                sh 'mvn -s settings.xml checkstyle:checkstyle'
                // uses checkstyple code analysis and suggest best practices and vulnerabilities.
                // note if do not add -s settings.xml then dependencies will be downloaded from maven instead
                //of Nexus repo.
            }
        }
    }
}
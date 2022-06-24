pipeline {
    agent any

    environment {
        GIT_PATH = "https://github.com/asemin08/ms-config.git"
        GIT_BRANCH = "main"
        GIT_PATH_config = "https://github.com/asemin08/spring-cloud-config.git"
    }

    stages {
        stage('Clean workspace'){
            steps{
            	cleanWs()
            }
        }
        stage('récupération du code source et récupération de la bonne branch') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "*/${GIT_BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[
                        url: "${GIT_PATH}"
                    ]]
                ])
            }
        }

        stage('récupération des fichier de conf') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "*/${GIT_BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[
                        url: "${GIT_PATH_config}"
                    ]]
                ])
            }
        }


        stage('Compile du projet') {
            steps {
                sh("mvn clean compile")
            }
        }


//         stage('test du projet') {
//             steps {
//                 sh("mvn test")
//             }
//             post {
//                 success{
//                     junit '**/target/surefire-reports/*.xml'
//                     jacoco execPattern: '**/target/**.exec', minimumClassCoverage: '70'
//                     emailext body: '$PROJECT_NAME - test # $BUILD_NUMBER - $BUILD_STATUS: Lien $BUILD_URL pour voir les résultat des test.', subject: '$PROJECT_NAME - Test ok ! # $BUILD_NUMBER - $BUILD_STATUS!', to: 'allan.semin@gmail.com'
//                 }
//                 failure{
//                     emailext body: '$PROJECT_NAME - test # $BUILD_NUMBER - $BUILD_STATUS: Lien $BUILD_URL pour voir les résultat des test.', subject: '$PROJECT_NAME - Test non ok ! # $BUILD_NUMBER - $BUILD_STATUS!', to: 'allan.semin@gmail.com'
//                 }
//             }
//         }
//
//         stage('Génération de la documentation du projet') {
//             steps {
//                 sh("mvn javadoc:javadoc")
//             }
//         }


        stage('packaging du projet') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh("mvn package sonar:sonar")
                }
            }
        }

        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Maven nexus archive') {
            steps {
                sh("mvn deploy -DskipTests")
            }
        }

        stage('lancement du build  pour créer l\'image') {
            steps {
                build job: 'BuildJavaImage', parameters: [string(value: 'openjdk', name: 'image'), string(value: '17-jdk-slim', name: 'tag'), string(value: 'verkeur08/ms-config', name: 'newImage'), string(value: 'eu.ensup', name: 'groupId'), string(value: 'microservice-test-config', name: 'artifactId')]
            }
        }

    }
}
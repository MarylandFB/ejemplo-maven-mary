import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        channel='D045UTVQ5PS'
        NEXUS_PASSWORD     = credentials('id-llamador')
    }
    stages {
        
        stage("Paso 0: Download Code and checkout"){
            steps {
                script{
                    checkout(
                            [$class: 'GitSCM',
                            //Acá reemplazar por el nonbre de branch
                            branches: [[name: "feature/newman" ]],
                            //Acá reemplazar por su propio repositorio
                            userRemoteConfigs: [[url: 'https://github.com/MarylandFB/ejemplo-maven-mary.git']]])
                }
            }
        }
     
        stage("Paso 1: Build && Test"){
            steps {
                script{
                    sh "echo 'Build && Test!'"
                    sh "./mvnw clean package -e"    
                }
            }
        }
  
        stage("Paso 2: Curl Springboot maven sleep 20"){
            steps {
                script{
                    sh "nohup bash ./mvnw spring-boot:run  & >/dev/null"
                   
                }
            }
        }
        stage("Paso 3: Detener Spring Boot"){
            steps {
                script{
                    sh '''
                        echo 'Process Spring Boot Java: ' $(pidof java | awk '{print $1}')  
                        sleep 20
                        kill -9 $(pidof java | awk '{print $1}')
                    '''
                }
            }
        }
           stage("Paso 4: Subir Artefacto a Nexus json"){
            steps {
                script{
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'maven-usach-ceres',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'json',
                                    filePath: 'ejemplo-maven.postman_collection.json'
                                ]
                            ],
                                mavenCoordinate: [
                                    artifactId: 'PruebaPostman',
                                    groupId: 'com.pruebapostman',
                                    packaging: 'json',
                                    version: '0.0.1'
                                ]
                            ]
                        ]
                }
            }
        }
           stage("Paso 5: Subir Artefacto a Nexus Jar"){
            steps {
                script{
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'maven-usach-ceres',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'jar',
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar'
                                ]
                            ],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020',
                                    groupId: 'com.devopsusach2020',
                                    packaging: 'jar',
                                    version: '0.0.1'
                                ]
                            ]
                        ]
                }
            }
        }        
        stage("Paso 6: Descargar Nexus"){
            steps {
                script{
                    sh ' curl -X GET -u admin:$NEXUS_PASSWORD "http://nexus:8081/repository/maven-usach-ceres/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }                                           
            }
        }
         stage("Paso 7: Levantar Artefacto Jar en server Jenkins"){
            steps {
                script{
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
          stage("Paso 8: Testear Artefacto - Dormir(Esperar 20sg) "){
            steps {
                script{
                    sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }
          stage("Paso 9: Testear Artefacto newman "){
            steps {
                script{
                    sh 'sleep 20 && newman run ejemplo-maven.postman_collection.json  -n 1  --delay-request 3000'
                }
            }
        }
        
        }
    
}
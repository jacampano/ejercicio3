//Configuración repositorio 
def credencialesGIT = "Gitlab"
def urlGIT = ""
def rama = ""


// Herramientas y servidores
def versionJDK = "JDK11"
def versionMaven = "Maven-3.8.6"
def dependencyCheckTool = "DC-7.3.0"
def sonarScannerTool = "SonarScanner"
def sonarToken = "SonarToken"
def instalacionSonar = "SonarLocal"

//Condiciones ejecución fases --
@Library('eslabones@main')
def realizarCompilacion = true
def analizarConSonar = true
def analizarConOWASP = true
def realizarPruebasUnitarias = false
def deployArtifact = false



pipeline {
    
    agent any
          
    stages {
        // Preparación tareas previas. Ejecución de tareas previas preparatorias de la ejecución de la Tubería.        
        stage('Inicialización')
        {
            steps{
               
                info()
                cleanWs()
                script {
                    urlGIT=env.gitlabSourceRepoHttpUrl
                    rama=env.gitlabBranch
                }
            }
        }

        stage('Descarga código fuente') {
            steps {
              
                ObtenerCodigoFuente(urlGIT,rama,credencialesGIT)
              
            }
                
  
        }
        
        stage('Compilación y empaquetado') {
              when {
                expression {
                    realizarCompilacion == true
                }
            }
          
                steps {
                    script {
                        
                        withEnv(["JAVA_HOME=${ tool versionJDK}"]) {
                            withMaven(maven: versionMaven) {

                            echo "--- Compilación y empaquetado ---"
                            echo "JAVA_HOME: $JAVA_HOME"
                            sh "mvn clean package"
                                       
                            }
                        }
         
                    }
                 
                }
            }
        
        
        //PRUEBAS DE COMPONENTES. Ejecución de pruebas individualizadas de los componentes que conforman el producto software. Estas pruebas no deben requerir el despliegue del producto.
        stage('Pruebas de Componentes'){
            stages {
                stage('Analisis estatico de código') {
                    when {
                        expression {
                           analizarConSonar == true
                          }
                     }

                    
                    steps {
                        catchError (buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                       
                            AnalisisSonar(versionJDK)
                
                        }
                }
                }

                stage('Umbral de Calidad') {

                     when {
                        expression {
                           analizarConSonar == true
                          }
                     }

                     steps {
                        catchError (buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            script {
                                QualityGate()
                            }
                            
                        }
                       
                    } 

                }

                stage('Analisis de dependencias') {
                    when {
                        expression {
                           analizarConOWASP == true
                          }
                     }

                    steps {
                        script {
                            try {
                                dependencyCheck additionalArguments: '''--format ALL''', odcInstallation: dependencyCheckTool
                            } catch (err) {
                                echo err.toString()
                            }
                        }

                    }

                    post {
                        success {
                            
                            echo "Acción a realizar si se ejecuta correctamente"

                        }
                    }
                }


                stage('Ejecución pruebas unitarias') {
                     when {
                        expression {
                           realizarPruebasUnitarias == true
                          }
                     }

  
                        steps {
                                                  
                            catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                                withEnv(["JAVA_HOME=${ tool versionJDK}"]) {
                                    withMaven(maven: versionMaven){

                                        sh "mvn test"
                                    }
                                }
                            }
 
                        }

                   post {
                        success {
                            echo "Acción a realizar si se ejecuta correctamente"
                         
                        }
                    }
                }

            }
           
            }//Fin pruebas de componentes
        

       

        //Aqui nueva fase que realiza el despliegue de un WAR.
        stage('Deploy') {
             steps {
                    script {
                        deploy adapters: [tomcat8(credentialsId: "tomcat", path: '', url: "http://localhost:9999/")], contextPath: "/pruebasWebApp", war: "**/*.war"
                    }
                }

        }

    }
        
        post {
            success {
                //dependencyCheckPublisher failedNewCritical: 5, failedNewHigh: 15, failedNewLow: 30, failedNewMedium: 15, failedTotalCritical: 10, failedTotalHigh: 30, failedTotalLow: 60, failedTotalMedium: 30, pattern: '**/dependency-check-report.xml', unstableNewCritical: 3, unstableNewHigh: 5, unstableNewLow: 25, unstableNewMedium: 15, unstableTotalCritical: 5, unstableTotalHigh: 20, unstableTotalLow: 50, unstableTotalMedium: 20
                echo "Acción cuando se completa con éxito"
            }
            aborted {
                
               echo "--- Aciones cuando se aborta ---"
                
            }
            failure {
                
                echo "-- Aciones cuando se falla ---"
              
                
            }

            always {
                echo "--- SE EJECUTA SIEMPRE ---"
            }
        } 
    }

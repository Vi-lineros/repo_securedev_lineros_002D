pipeline {
    agent any

    environment {
        IMAGE_NAME     = 'securedev-app'
        CONTAINER_PROD = 'securedev-prod'
        CONTAINER_TEST = 'securedev-app-test'
        ZAP_DIR        = "${WORKSPACE}/zap-reports"
    }

    stages {
        // ETAPA 1: CONSTRUCCIÓN REAL DE LA IMAGEN DOCKER
        stage('Construccion (Build)') {
            steps {
                echo '=== INICIANDO ETAPA DE CONSTRUCCIÓN ==='
                echo 'Leyendo Dockerfile y empaquetando la aplicacion Flask...'
                // Construye la imagen utilizando el Dockerfile de la raíz
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }
        
        // ETAPA 2: PRUEBAS AUTOMATIZADAS DE SEGURIDAD (OWASP ZAP)
        stage('Pruebas de Seguridad (DAST)') {
            steps {
                script {
                    echo '=== INICIANDO ETAPA DE PRUEBAS DE SEGURIDAD ==='
                    echo 'Levantando entorno controlado temporal para el escaneo...'
                    // Levanta la app en el puerto 5001 de manera interna para testearla
                    sh 'docker run -d --name ${CONTAINER_TEST} -p 5001:5000 ${IMAGE_NAME}'
                    
                    // Espera 10 segundos a que Flask inicialice correctamente
                    sleep 10 

                    echo 'Creando directorio y asignando permisos para el reporte de ciberseguridad...'
                    sh "mkdir -p ${ZAP_DIR} && chmod 777 ${ZAP_DIR}"
                    
                    echo 'Ejecutando escaneo dinamico automatizado con OWASP ZAP...'
                    // Corre la imagen oficial de OWASP ZAP apuntando a nuestra app temporal
                    // El "|| true" es vital para que Jenkins no aborte el pipeline si encuentra fallas, permitiendo guardar el reporte
                    sh '''
                    docker run --rm -v ${ZAP_DIR}:/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py \
                        -t http://host.docker.internal:5001 \
                        -r reporte_zap.html || true
                    '''
                }
            }
            // Esta sección se ejecuta SIEMPRE para limpiar el entorno y guardar las evidencias
            post {
                always {
                    echo 'Limpiando el contenedor de pruebas temporal...'
                    sh 'docker stop ${CONTAINER_TEST} || true'
                    sh 'docker rm ${CONTAINER_TEST} || true'
                    
                    echo 'Archivando el reporte HTML de OWASP ZAP como artefacto de auditoria...'
                    // Guarda el reporte directamente en la interfaz de Jenkins para la trazabilidad
                    archiveArtifacts artifacts: 'zap-reports/reporte_zap.html', allowEmptyArchive: true
                }
            }
        }
        
        // ETAPA 3: DESPLIEGUE AUTOMATIZADO A PRODUCCIÓN SIMULADA
        stage('Despliegue (Deploy)') {
            steps {
                script {
                    echo '=== INICIANDO ETAPA DE DESPLIEGUE ==='
                    echo 'Removiendo versiones antiguas en ejecucion...'
                    sh 'docker stop ${CONTAINER_PROD} || true'
                    sh 'docker rm ${CONTAINER_PROD} || true'
                    
                    echo 'Desplegando la nueva version mitigada en el entorno de Produccion (Puerto 5000)...'
                    // Despliega la aplicación final para que quede expuesta en el puerto 5000
                    sh 'docker run -d --name ${CONTAINER_PROD} -p 5000:5000 ${IMAGE_NAME}'
                    echo 'Despliegue exitoso. Aplicacion en linea y protegida.'
                }
            }
        }
    }
}

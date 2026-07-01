pipeline {
    agent any

    stages {
        stage('Construccion') {
            steps {
                echo 'Construyendo la imagen Docker de la aplicacion...'
                // sh 'docker build -t securedev-app .'
            }
        }
        stage('Pruebas') {
            steps {
                echo 'Ejecutando analisis de seguridad y pruebas...'
            }
        }
        stage('Despliegue') {
            steps {
                echo 'Desplegando la aplicacion en el entorno local...'
            }
        }
    }
}

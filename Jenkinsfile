pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              serviceAccountName: jenkins # A SA configurada com Workload Identity
              containers:
              - name: builder
                image: node:18-alpine
                command: ['sleep', 'infinity']
              - name: uploader
                image: google/cloud-sdk:alpine
                command: ['sleep', 'infinity']
            '''
        }
    }

    environment {
        BUCKET_NAME = 'my-portfolio-assets'
        PROJECT_ID = 'natour' // Nome único deste projeto
    }

    stages {
        stage('Build SASS') {
            steps {
                container('builder') {
                    // Instala dependências e compila SASS para CSS
                    sh 'npm install' 
                    sh 'npm run build:sass' // Deve gerar, ex: ./dist/style.css
                }
            }
        }

        stage('Deploy to GCS') {
            steps {
                container('uploader') {
                    script {
                        // Estratégia de Versionamento:
                        // 1. Deploy da versão específica (baseada no GIT TAG ou BUILD_NUMBER)
                        def version = env.BUILD_TAG ?: "v${env.BUILD_NUMBER}"
                        
                        sh """
                          gsutil -m rsync -r ./dist gs://${BUCKET_NAME}/projects/${PROJECT_ID}/${version}
                        """

                        // 2. Atualizar o ponteiro 'latest' (Opcional)
                        sh """
                          gsutil -m rsync -r ./dist gs://${BUCKET_NAME}/projects/${PROJECT_ID}/latest
                        """
                        
                        // 3. (Opcional) Atualizar um arquivo manifest.json no bucket
                        // Isso permite que o Angular saiba quais versões existem para listar num combobox
                    }
                }
            }
        }
    }
}
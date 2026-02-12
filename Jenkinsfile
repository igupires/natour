pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
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
        stage('Build & Organize Assets') {
            steps {
                container('builder') {
                    echo ' 1. Instalando dependências...'
                    sh 'npm install'

                    echo ' 2. Criando pasta de build...'
                    sh 'rm -rf dist && mkdir dist'


                    echo ' 3. Construindo os assets...'
                    sh 'npm run build'

                    echo ' 4. Copiando arquivos estáticos...'
                    sh 'cp index.min.html dist/index.html'
                    sh 'cp -r css dist/css/'
                    sh 'cp -r img dist/img/'

                    echo ' BUILD SNAPSHOT: '
                    sh 'ls -R dist/'
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
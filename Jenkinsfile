pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_SITE_ID = credentials('netlify-site-id')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🏗️ Building the project..."
                    sh '''
                    npm install
                    npx react-scripts build'''
                }
            }
            post {
                success {
                    echo "✅ Build Successful! 🎉"
                }
                failure {
                    echo "❌ Build Failed! Check logs for details."
                }
            }
        }

        // Run tests and deploy in parallel
        stage('Test and Deploy') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        script {
                            echo "🔬 Running tests..."
                            sh 'npm test'
                        }
                    }
                    post {
                        success {
                            echo "✅ Test Successful! 🎉"
                        }
                        failure {
                            echo "❌ Test Failed! Check logs for details."
                        }
                    }
                }

                stage('Deploy to Netlify') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        script {
                            echo "🚀 Deploying to Netlify..."
                            sh '''
                            npx netlify deploy --prod --dir=build \
                            --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                            '''
                        }
                    }
                    post {
                        success {
                            echo "✅ Deployment Successful! 🎉"
                        }
                        failure {
                            echo "❌ Deployment Failed! Check logs for details."
                        }
                    }
                }
            }
        }

        stage('Post Deploy') {
            agent any
            steps {
                script {
                    echo "🔍 Monitoring server resources during the test..."
            
                    try {
                        sh '''
                            echo "Top 10 processes by memory usage:" > resource_report.txt
                            ps aux --sort=-%mem | head -n 10 >> resource_report.txt
                            
                            echo "\nMemory usage:" >> resource_report.txt
                            free -h >> resource_report.txt
                            
                            echo "\nSystem performance stats (vmstat):" >> resource_report.txt
                            vmstat 1 5 >> resource_report.txt
                        '''
                    } catch (e) {
                        echo "Error monitoring server resources: ${e}"
                    }
                }
            }
            post {
                success {
                    echo "✅ Resource monitoring completed successfully! Here are the results:"
                    sh 'cat resource_report.txt'
                }
                failure {
                    echo "❌ Resource monitoring encountered an error!"
                }
            }
        }
    }
}

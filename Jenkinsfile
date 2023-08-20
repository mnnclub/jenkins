pipeline {
    // 스테이지 별로 다른 거다..
    agent any

    // 젠킨스 빌드 트리거의 poll scm 옵션켜야 되고 그안에 스케쥴 설정은 해도 안먹어서 여기다 해둬야 함 - 라고 2년전에 얘기했었음
    triggers {
        pollSCM('H/30 * * * *')
    }

    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
    }

    stages {
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            
            steps {
                echo '#####[1/6] Clonning Repository #####'

                // git credentialsId: 'token for Jenkins', url: 'https://github.com/mnnclub/jenkins'
                git url: 'https://github.com/mnnclub/jenkins',
                    branch: 'master',
                    credentialsId: 'token for Jenkins'
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo '##### Successfully Cloned Repository #####'
                }

                always {
                  echo "i tried..."
                }

                cleanup {
                  echo "##### after all other post condition #####"
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo '#####[2/6] Deploying Frontend #####'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://jihyeonho
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo '##### Successfully Cloned Repository #####'

                  mail  to: 'mnnclub3@gmail.com',
                        subject: "Deploy Frontend Success",
                        body: "Successfully deployed frontend!"

              }

              failure {
                  echo '##### I failed :( #####'

                  mail  to: 'mnnclub3@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              echo '#####[3/6] Linting Backend #####'
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo '#####[4/6] Test Backend #####'

            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo '#####[5/6] Build Backend #####'


            // docker build . -t server --build-arg env=${PROD}
            dir ('./server'){
                sh """
                docker build . -t server
                """
            }
          }

          post {
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo '#####[6/6] Build Backend #####'

            // 최초실행때는 실행중인 도커가 없어서 제거하되 2번째 이후부터는 필요함: docker rm -f $(docker ps -aq)
            // 위에 빌드백엔드 스테이지에서 도커 빌드 server 로 해줬으니까 그이름 그대로 아래에.
            dir ('./server'){
                sh '''
                docker run -p 80:80 -d server
                '''
            }
          }

          post {
            success {
              mail  to: 'mnnclub3@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
}

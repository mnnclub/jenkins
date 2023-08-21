pipeline {
    // 스테이지 별로 다른 거다..
    agent any

    // 젠킨스 빌드 트리거의 poll scm 옵션켜야 되고 그안에 스케쥴 설정은 해도 안먹어서 여기다 해둬야 함 - 라고 2년전에 얘기했었음
    triggers {
        pollSCM('0 0 * * *')
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
        // aws 에서 뭔가 토큰값을 인자로 받야아 허용하게끔 기본적으로 되어있는듯함 그냥 접속하면 403 차단됨, 접속은 aws s3 에서 주소복사 해야함, 아래주소~
        // https://jihyeonho.s3.ap-northeast-2.amazonaws.com/index.html?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEPz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLW5vcnRoZWFzdC0yIkYwRAIgT0BZ3pvrqNeP1FL1fJhjbwmivac6oRgNDDqwMRgOkRcCIAWSF3XTRy9%2BNYoJX8Z8Rof5aGqS4WOF1YPTmysJTR%2BDKu0CCLX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQABoMNDYxNTY0MTIyODY5IgzP9HLJ3ThfZtt240wqwQJK8ISIB4nuhdHaVVpri9QgiZHy5D%2F%2BIlz9CT8AIap4pshkQP8xzf1Mw6afWLYVRk%2Bq1nGL5PreR2T8GwYF0dzVoV1KYJZuPCpuo1uIoDDsPhRMZBVYyhHmmH3Dtq2RqwXsa3OILQ4tk6Qy2%2BiGi2XCf8quxTp%2B5X7UtVGQWFgSqsSWI06kkdNSqBpe6hox2%2FYTdBQVjNQRW0IydMD%2F3cq%2FsaVIVTb5tn6ix8Yxo14CjkspL7SJ%2FvaAx9AtUX0mfiwE%2FtOw91q%2F24MDGPhmeP93qIzONROUiiDghBZCGWFgGRuln4LLFsmny08Xi4VPrSYAEPbpO1Gnh2lI6LCCBGxJ8kNMbj09dwxOzO1lM3rIhr%2FV2JKE5cMXG6pEOB9HxpJ3WynFMdyT2kq549UccWw1YjIYObyck%2BVblowns1ZmA9kwpd2FpwY6tAKK8AFzfJPd0NJajAj6TKCOtlYX0yqEgiwIIx5D1uWAse2ddNBl%2F8mYyLQ4kdi0TecDXkH%2FkA0UmzHZh33As2zFZZZOppEjLxIPvism%2BfSBt%2BuehCUHJ4AFHTA7GX%2BmjKVcwIWg5obltg3ZJ%2FUawd252qBwePjVCNZV2k3Fin0GxNummNU8xwEult9XKmTbww%2BTbNd%2BUWAJwjXZvA62%2BGwvE30O%2B%2Bv%2BMpvX4QPoZ%2BUr1ZrGH%2BM%2B5L3t1O1n5aZCIqA4SUJmeLQKOADlS7%2Fz9sz5fMWfXUwE%2FOg%2Fc43EZqAz%2B02eYx7TcB96WwXma4mS2bL0VlUXHnJlrUTxB1sTh%2FdgHoVD1i9NA2nwYgCsI7sGiMqddLw5RdEH1EbiGzPBvPoMddN%2BykmXRoVpG7ZeaiVSlPjwoQ%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230820T093221Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAWW525S3263KSYNP7%2F20230820%2Fap-northeast-2%2Fs3%2Faws4_request&X-Amz-Signature=21579bdbdf348cf2ffd28bc2007223ac6936696721c1e708def246c337d377fc

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
            // 도커 컨테이너가 1개이상 존재할경우에만 삭제하도록 추가로 만들어둠
            dir ('./server'){
                sh '''
                [ $(docker ps -aq |wc -l) -ge 1 ] && docker rm -f $(docker ps -aq)
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

pipeline 
{
    agent 
    {
        label 'docker-node'
    }
    environment
    {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    stages 
    {
        stage('Clone') 
        {
            steps 
            {
                git branch: 'main', url: 'http://192.168.69.152:3000/stoimenovrado/devopshomework'
            }
        }
        stage('Docker compose')
        {
            steps
            {
                sh '''
                    docker container rm -f do-hw_web_1 do-hw_db_1 || true
                    docker-compose up -d
                    '''
            }
        }
        stage('Test')
        {
            steps
            {
                script 
                {
                    echo 'Test #1 - reachability'
                    sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080) | grep 200'
                    
                    sleep 10

                    echo 'Test #2 - is the DB up?'
                    sh "curl http://localhost:8080 | grep Plovdiv"

                }
            }
        }
        stage('CleanUp')
        {
            steps
            {
                sh 'docker container rm -f do-hw_web_1 do-hw_db_1 || true'
            }
        }
        stage('Login') 
        {
            steps 
            {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push') 
        {
            steps 
            {
                sh 'docker image build img-web stoimenovrado/bgapp-web -f Dockerfile.web'
                sh 'docker push stoimenovrado/bgapp-web'
                sh 'docker image build img-db stoimenovrado/bgapp-db -f Dockerfile.db'
                sh 'docker push stoimenovrado/bgapp-db'
            }
        }
        stage('Docker compose PROD')
        {
            steps
            {
                sh '''
                    docker container rm -f do-hw_web-prod_1 do-hw_db-prod_1 || true
                    docker-compose -f docker-compose-prod.yaml up -d
                    '''
            }
        }
    }
}
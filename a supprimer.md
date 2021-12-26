stages{

       stage ('Build Image'){
           agent any
           steps {
               withCredentials([sshUserPrivateKey(credentialsId: "kader_ec2_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{
                            sh '''

                            ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker build -t $USERNAME/$IMAGE_NAME:$BUILD_TAG .
                            
                            '''
               
               }
           }
       }
    }
}

       stage ('Run test container') {
           agent any
           steps {
               withCredentials([sshUserPrivateKey(credentialsId: "kader_ec2_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{
                            sh '''
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker stop $CONTAINER_NAME || true
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker rm $CONTAINER_NAME || true
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker run --name $CONTAINER_NAME -d -p 5000:80 $USERNAME/$IMAGE_NAME:$BUILD_TAG                            
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} sleep 5
                            '''
               
               }
           }
       }
    }
}

       stage ('Test application') {
           agent any
           steps {

               script{
                   sh '''
                       ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} curl http://localhost:5000 | grep -iq "Dimension"
                   '''
               }
           }
       }
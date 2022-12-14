pipeline 
{
    
    
    agent 
    {
        label 'maven'
    }
    
    stages 
    {
        
        
        
        stage('Build')
        {
            steps
            {
                echo 'Building...'
                sh 'mvn clean package'
            }
        }
        
        
        stage('Create Container Image')
        {
            steps 
            {
                echo 'create container image..'
                script
                {
                    openshift.withCluster()
                    {
                        openshift.withProject("new-pro")
                        {
                            def buildConfigExists = openshift.selector("bc", "onlinebookstore").exists()
                            
                            if(!buildConfigExists)
                            {
                                openshift.newBuild("--name=onlinebookstore", "--docker-image=registry.redhat.io/jboss-webserver-3/webserver31-tomcat8-openshift", "--binary")
                            }
                            
                            openshift.selector("bc", "onlinebookstore").startBuild("--from-file=target/dependency/webapp-runner.jar", "--follow")
                            
                        }
                    }
                }
            }
        }
        
        
        
        
        stage('Deploy')
        {
            steps
            {
                echo 'Deploying....'
                script
                {
                    openshift.withCluster()
                    {
                        openshift.withProject("new-pro")
                        {
                            def deployment = openshift.selector("dc", "onlinebookstore")
                            
                            if(!deployment.exists())
                            {
                                openshift.newApp('onlinebookstore', "--as-deployment-config").narrow('svc').expose()
                            }
                            
                            timeout(5)
                            {
                                openshift.selector("dc", "onlinebookstore").related('pods').untilEach(1)
                                {
                                    return (it.object().status.phase == "Running")
                                }
                            }
                            
                            
                        }
                    }
                }
            }
        }
        
        
        
        
    }
    
    
}

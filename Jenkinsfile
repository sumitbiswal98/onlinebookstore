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
                        openshift.withProject("project-cicd")
                        {
                            def buildConfigExists = openshift.selector("bc", "cicd").exists()
                            
                            if(!buildConfigExists)
                            {
                                openshift.newBuild("--name=cicd", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
                            }
                            
                            openshift.selector("bc", "cicd").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow")
                            
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
                        openshift.withProject("project-cicd")
                        {
                            def deployment = openshift.selector("dc", "cicd")
                            
                            if(!deployment.exists())
                            {
                                openshift.newApp('cicd', "--as-deployment-config").narrow('svc').expose()
                            }
                            
                            timeout(5)
                            {
                                openshift.selector("dc", "cicd").related('pods').untilEach(1)
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
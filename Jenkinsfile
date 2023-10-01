 def Build_pass=true
 def Deploy_to_Dev_pass=true
  def UnitTest_pass=true
   def Deploy_to_QA_pass=true
   def E2e_tests_pass= true

pipeline {
    agent any
      tools { 
      maven 'maven3' 
      jdk 'java 8' 
    }
        environment{
          remoteServerDev = 'dev'
          remoteServerQA = 'ssh qa'
          remoteServerProd = 'ssh production'
         build = "sh /home/ec2-user/build.sh"
         deploy = "sh /home/ec2-user/deploy.sh"
}
    stages {
        stage('Build') {
            steps {
                script {
                    try{
                        //trigga o pipeline no momento em que um push é feito no codigo da aplicacao
                        checkout([$class: 'GitSCM',  poll: true, branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/clark-ewerton/Online_Fruits_And_Veggies_DEVOPS.git']]])
            // Executar comandos SSH usando o plugin "Publish Over SSH"
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: remoteServerDev, // O nome do servidor SSH configurado
                        transfers: [
                            sshTransfer(
                                sourceFiles: '',
                                removePrefix: '',
                                remoteDirectory: '',
                                execCommand: build // O comando que você deseja executar
                            )
                        ]
                    )
                ]
            )
        }
        catch (Exception e){
        Build_pass = false
        }
        }
            }
        }
        stage('deploy_to_dev') {
           steps {
                script {
            // Executar comandos SSH usando o plugin "Publish Over SSH"
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: remoteServerDev, // O nome do servidor SSH configurado
                        transfers: [
                            sshTransfer(
                                sourceFiles: '',
                                removePrefix: '',
                                remoteDirectory: '',
                                execCommand: deploy // O comando que você deseja executar
                            )
                        ]
                    )
                ]
            )
       
        }}
        }
        stage('Unit Tests') {
             steps {
     
     script{
	 if(Deploy_to_Dev_pass){
	 
	 try{
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/clark-ewerton/UnitTests_Devops.git']]])
		bat "mvn clean test"
		}
		catch(Exception e){
		UnitTest_pass=false
     }
     
}
	 
    }
}
    }
      stage('deploy_to_qa') {
           steps {
                script {
                    if(UnitTest_pass){
                 
				 try{
            // Executar comandos SSH usando o plugin "Publish Over SSH"
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: remoteServerQA, // O nome do servidor SSH configurado
                        transfers: [
                            sshTransfer(
                                sourceFiles: '',
                                removePrefix: '',
                                remoteDirectory: '',
                                 execCommand: "sh -c \"${build} && ${deploy}\"" // Use as variáveis build e deploy corretamente
                            )
                        ]
                    )
                ]
            )
       
        }
catch(Exception e){
Deploy_to_QA_pass=false
}}
        }  
        
        
    }}
        
        
        stage('E2e_tests'){
 steps {
 script{
 
 if(Deploy_to_QA_pass){
 
 try{
 
 checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/clark-ewerton/E2E_tests_Devops.git']]])
bat 'mvn clean test -DsuiteXmlFile=testng.xml'

}
catch(Exception e){
E2e_tests_pass=false
}

}

}
}
}

stage('deploy_to_prod'){
steps {
                 script
                 {
                 
               if(E2e_tests_pass){
			    // Executar comandos SSH usando o plugin "Publish Over SSH"
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: remoteServerProd, // O nome do servidor SSH configurado
                        transfers: [
                            sshTransfer(
                                sourceFiles: '',
                                removePrefix: '',
                                remoteDirectory: '',
                                 execCommand: "sh -c \"${build} && ${deploy}\"" // Use as variáveis build e deploy corretamente
                            )
                        ]
                    )
                ]
            )

}
}
}
}
    }

    post {
        failure {
            echo 'O pipeline falhou!'
            // Adicione ações ou notificações adicionais em caso de falha
        }
    }
}

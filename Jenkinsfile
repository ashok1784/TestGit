env.JAVA_HOME = "/Users/i055812/Documents/Hybris/sapjvm_8/jre/"

//def POM_LIB="""${WORKSPACE}/Selenium/lib/"""
def MVN_BIN="""/Users/i055812/Documents/Hybris/HPE/6712/framework/resouces/apache-maven-3.5.0/bin"""
def cleanbuild=""". ./setantenv.sh && ant clean all"""
def codecheck=""". ./setantenv.sh && ant yunitinit unittests sonarcheck"""
def Repository_Path="Hybris_DEV"
def Repository_Url="http://localhost:8081/artifactory/"
def Repository_Username="admin"
def Repository_Password="Welcome1"
def Artifact_Repository_Max_Build_Retention_Number="2"
def Artifact_Repository_Max_Build_Retention_Days="2"

node {
    def WORKSPACE = pwd()
    //def generatezips=""". ./setantenv.sh && ant production -Dproduction.create.zip=true -Dproduction.output.path=pwd()"""
  stage ('Checkout Code from GIT')  
  {
    echo 'Checkout Code from Source Code Repository'
    //checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'b84b3938-ddea-4b04-83d7-c5e8ecd12d9c', url: 'https://github.com/pfizer/primehybrisext']]])
  }
  stage('Build') 
  {
                try {
                        dir('/Users/i055812/Documents/Hybris/HPE/6712/hybris/bin/platform'){
                        sh "$cleanbuild"
                        echo 'Executing Ant clean all'
                        }
                    }
                catch(err){
                        buildFailure = true;
                        throw err
                        currentBuild.result = 'FAILURE'
                }   
                }
  stage('Executing Unit Tests, Code Coverage, Static Code Checks & Selenium') {
    parallel (
     "Executing Unit Tests, Code Coverage & Static Code Checks":{
                    try 
                    {
                          dir('/Users/i055812/Documents/Hybris/HPE/6712/hybris/bin/platform'){ 
                               
                            sh "$codecheck"
                            
                            echo 'Executing ant junittests'
                            }
                    }catch(err)
                    {
                        buildFailure = false;
                        throw err
                        currentBuild.result = 'FAILURE'
                    }
                }, 
    "Executing Selenium Tests":{     
            //env.CLASSPATH=.:$POM_LIB:${MVN_BIN}
              //  dir('$SVN_POM_DIR'){
                //${MVN_BIN}/mvn clean test }
                echo 'Hello, Maven'
                echo 'Hello, Selenium'
}
)

            }
  stage('Generate Production Zips') {
                    try 
                    {
                          dir('/Users/i055812/Documents/Hybris/HPE/6712/hybris/bin/platform'){ 
                           
                            echo 'Executing ant production'
                            sh ". ./setantenv.sh && ant production -Dproduction.output.path=${WORKSPACE} -D-Dproduction.create.zip=true"
                            }
                    }catch(err)
                    {
                        buildFailure = true;
                        throw err
                        currentBuild.result = 'FAILURE'
                    }
                } 
         
     
    stage('Publish JUnit test results') {

    def exists = fileExists 'TESTS-TestSuites.xml'

if (exists) {
    echo 'File Exists. Deleting old file'
    fileOperations([fileDeleteOperation(excludes: '', includes: 'TESTS-TestSuites.xml')])
} else {
    echo 'No'
}

    dir("/Users/i055812/Documents/Hybris/HPE/6712/hybris/log/junit/") {
    fileOperations([fileCopyOperation(excludes: '', flattenFiles: true, includes: '*.xml', targetLocation: "${WORKSPACE}")])
findFiles glob: '*.xml'
}

      junit 'TESTS-TestSuites.xml'
  } 
stage('Upload Files to Artifactory') {
                          
                            rtServer (
                                id: "Artifactory-1",
                                url: "${Repository_Url}",
                                username: "${Repository_Username}",
                                password: "${Repository_Password}"
                            )
                            rtUpload (
                                serverId: "Artifactory-1",
                                spec:
                                    """{
                                      "files": [
                                        {
                                          "pattern": "*hybrisServer-*.zip",
                                          "target": "${Repository_Path}/Hybris_Jenkins_V_${BUILD_NUMBER}/"
                                        }
                                     ]
                                    }"""
                            )     
                            
    }                         
}
#!groovy

properties([
	[$class: 'GithubProjectProperty', displayName: 'oap', projectUrlStr: 'https://github.com/xpdable/pipeline-as-code-poc/'], 
	pipelineTriggers([githubPush()])
])

list_general_stage_status=[]

node('inhouse-awscli'){

    stage('scm'){
        
        checkout([$class: 'GitSCM', 
        	branches: [[name: '*/master']], 
        	doGenerateSubmoduleConfigurations: false, 
        	extensions: [], 
        	submoduleCfg: [], 
        	userRemoteConfigs: [
        		[credentialsId: 'pool-id-a133-s-itioap', 
        		name: 'origin', 
        		url: 'https://github.com/xpdable/pipeline-as-code-poc/']
        	]])
    }//end of scm


    //Parse and generate stage
    def configJSON = readJSON file: 'execution.json'
    stage_map = [:]
    //parse json to stage map
    /**
    *   [1:[stage1,stage2],2:[stage3],3:[stage4,stage5,stage6],4:[stage7], .... ]
    *   def m = [:]
    *   Then, you can see Groovy by default makes a LinkedHashMap
    *    assert m.getClass().name == 'java.util.LinkedHashMap'
    */
    println "[DEBUG] main-pipeline : after readJSON"
    for ( action in configJSON.actions ){
        if (stage_map.containsKey(action.order)){
            stage_map.get(action.order).add(action)
        }else{
            def parallel_action_list = []
            parallel_action_list.add(action)
            stage_map.put(action.order,parallel_action_list)
        }
    }
    println stage_map
    println "[DEBUG] main-pipeline : after parse actions into LinkedHashMap"
    stage_map_keyset = stage_map.keySet()
    for ( int i = 0 ; i < stage_map_keyset.size(); i++ ){
        def stage_source_list = []
        stage_source_list = stage_map.get(stage_map_keyset[i])
        if ( stage_source_list.size() > 1 ){
            //parallel
            println "[DEBUG] main-pipeline : starts parallel jobs"
            parallel_map = [:]
            for(int j = 0; j < stage_source_list.size() ; j++){
                parallel_map.put(stage_source_list[j].stage, parallelStageWrapper(stage_source_list[j]))
            }
            parallel parallel_map
        }else{
            //series
            println "[DEBUG] main-pipeline : starts single jobs"
            def result = stageGenerator(stage_source_list[0])
        }
    }

}//end of node 

def parallelStageWrapper(Object action){

    return{
        stageGenerator(action)
    }

}

def stageGenerator(Object action){
    
        def stageName = action.stage
        def stageType = action.type
        def stageSource = action.source
        def params = action.parameter
        def period = 0
        try {
            period = action.quietPeriod
        }catch(e){
            println "no quietPeriod defined, use default 0"
        }
        def wait
        try{
            if (action.wait == 1){
                wait = true 
            }else{
                wait = false
            }
        }catch(e){
            //do nth
            wait = true
        }
        def propagate
        try{
            if (action.propagate == 1){
                propagate = true 
            }else{
                propagate = false
            }
        }catch(e){
            //do nth
            propagate = true
        }

        stage(stageName){
            println "[DEBUG] main-pipeline : start stages from map"
            //Invoke Jenkins Job / Pipeline
            if (stageType == "job"){  //External API Job
                if (stageSource.startsWith("http")){
                    println "[DEBUG] main-pipeline : start API job"
                    //Do API invoke
                    //TODO 
                }else{  //Internal Jenkins Job
                    println "[DEBUG] main-pipeline : start internal job ${stageSource}"
                    build job: stageSource, 
                          parameters: buildJobParameter(params), 
                          quietPeriod: period, 
                          wait: wait,
                          propagate : propagate
                }
            }else if(stageType == "script"){ //Simple Shell
                println "[DEBUG] main-pipeline : start script job "
                sh "${stageSource} ${params}"
            }else if(stageType == "importSource"){ //Import Groovy
                println "[DEBUG] main-pipeline : start groovy job : loading ... ${stageSource}"
                def imported_script = load "${stageSource}"
                result = imported_script.main(params)  //Remember add "return this" at the end ; only main() accept for now TODO
            }else if(stageType == "sharedFunction"){
                println "[DEBUG] main-pipeline : start shared function "
                def result = "${stageSource}"(params)
            }else{
                //TODO 
            }//end of if type
        }//end of stage

}//end of stageGenerator def

/**
*  e.g.  [{"type":"string","name":"version","value":""},
*         {"type":"string","name":"targetIP","value":"10.200.29.231"},
*         {"type":"string","name":"waitreport","value":"ScanAndWaitReport"}]
*   params -> ArrayList<LinkedHashMap>
*   expected return:
*   [ string(name: 'version', value: ''),  string(name: 'targetIP', value: '10.200.29.231'), string(name: 'waitreport', value: 'ScanAndWaitReport')]
*
**/
def buildJobParameter(Object params){
    param_list = []
    for ( int i = 0; i < params.size(); i++){
        if( params[i].type == "string" ){
            param_list.add( string( name: params[i].name, value: params[i].value ) )
        }else if ( p.type == 'other'){
            //TODO not supported
        }else{
            //TODO not supported
        }
    }
    println "[DEBUG] main-pipeline : buildJobParameter : param_list = ${param_list} "
    return param_list
}//end of buildJobParameter def


def sharedFunctionDemo(Object p){
    println p
}

// stage('manual-entry'){
//     build job: 'Utilities/manual-approval-poc', quietPeriod: 0   
// }

// stage('sonarqube-pipeline'){
//     build job: 'Utilities/SonarQube-OAP', parameters: [string(name: 'version', value: '1.4.0'), string(name: 'branch', value: 'sprint')], quietPeriod: 0
// }

// stage('qualys-pipeline'){
//     build job: 'Utilities/QualysScan', parameters: [string(name: 'version', value: ''),  (name: 'targetIP', value: '10.200.29.231'), string(name: 'waitreport', value: 'ScanAndWaitReport')], quietPeriod: 0
// }

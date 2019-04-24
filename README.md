# pac-poc
Pipeline as code PoC

## Mechanism
Jenkinsfile will parse execution.json and generate the pipeline stage dynamically according to actions:\[] 

## Execution.json Configuration
Please refer to example for understanding: 
- [poc excution](https://git.daimler.com/ITT-China/pac-poc/blob/master/execution.json) 
- [parallel-stage-example](https://git.daimler.com/ITT-China/pac-poc/blob/master/execution-parallel.json) 
- [series-style-example](https://git.daimler.com/ITT-China/pac-poc/blob/master/execution-series-demo.json) 

```
 "actions": [
        {
            "stage":"stage-internal-job",
            "order":1,
            "type":"job",
            "source":"Utilities/manual-approval-poc",
            "parameter":[],
            "wait":1,
            "quietPeriod":0,
            "propagate":1
        }
      ]
```        
* **`stage`** is the stage display name
* **`order`** is desired exeuction order, use same integar if you want do stage in parallel
* **`type`**, **`source`** and **`parameter`** work together
### `type`:`job`  
  Invoke an internal Jenkins job //TODO : API call/callback with external Jenkins in next release
  - **`type`** : `job` for now only internal Jenkins job is supported
  - **`source`** : source of a `job` : the job name. e.g. `Utilities/QualysScan`
  - **`parameter`** : parameter of a `job` : only string is implemented. e.g. 
  
    ```
        "parameter":[
              {"type":"string","name":"version","value":""},
              {"type":"string","name":"targetIP","value":"10.200.29.231"}, 
              {"type":"string","name":"waitreport","value":"ScanAndWaitReport"}
              ]
    ```
  - **`*wait`** : if wait the job complete, only for `type`:`job`
  - **`*quietPeriod`** : delay start time of the job, only for `type`:`job`, default to `1` as `true`
  - **`*propagate`** : if raise the error from the job, only for `type`:`job`, default to `1` as `true`
  
### `type`:`script`   
  Execute simple shell command or scripts from SCM
  - **`type`** : `script` simple execute shell script
  - **`source`** : source of a `script` : the shell command. e.g. `ls`
  - **`parameter`** : parameter of a `script` : the sufix of your command. e.g. '-li | grep xxx'
  
### `type`:`importSource` 
  Invoke groovy or other Jenkins pipeline file
  - **`type`** : `importSource` 
  - **`source`** : source of a `importSource` : the groovy script path and name, `main()` should be defined
  - **`parameter`** : parameter of a `importSource` : the parameter of `main(parameter)`
  
### `type`:`sharedFunction` 
  Invoke common shared function in the current Jenkinsfile, e.g. execute AWS CF ChanageSet
  - **`type`** : `sharedFunction` 
  - **`source`** : source of a `sharedFunction` : the function and name
  - **`parameter`** : parameter of a `sharedFunction` : the parameter of the function
  
## Execution
Is invoked by Jenkins via webhook, read more about Jenkins Pipeline [here](https://jenkins.io/doc/book/pipeline/)

### Example
- Series pipeline
![execution-series-demo.json](https://git.daimler.com/ITT-China/pac-poc/blob/master/iac/s.jpg)
- Parallel pipeline
![execution-parallel.json](https://git.daimler.com/ITT-China/pac-poc/blob/master/iac/p.jpg)

  

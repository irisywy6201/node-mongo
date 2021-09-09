My Docker Development Workflow: Code, Build, Push, Run

## CI/ CD

- 用gitlab + Laravel來幫助了解CICD
    - [https://medium.com/@nick03008/gitlab-ci-入門筆記-單元測試篇-156455e2ad9f](https://medium.com/@nick03008/gitlab-ci-%E5%85%A5%E9%96%80%E7%AD%86%E8%A8%98-%E5%96%AE%E5%85%83%E6%B8%AC%E8%A9%A6%E7%AF%87-156455e2ad9f)
    - [https://medium.com/@nick03008/教學-gitlab-ci-入門實作-自動化部署篇-ci-cd-系列分享文-cbb5100a73d4](https://medium.com/@nick03008/%E6%95%99%E5%AD%B8-gitlab-ci-%E5%85%A5%E9%96%80%E5%AF%A6%E4%BD%9C-%E8%87%AA%E5%8B%95%E5%8C%96%E9%83%A8%E7%BD%B2%E7%AF%87-ci-cd-%E7%B3%BB%E5%88%97%E5%88%86%E4%BA%AB%E6%96%87-cbb5100a73d4)
- Azure+pipline(30day系列)
    - [https://ithelp.ithome.com.tw/articles/10206169](https://ithelp.ithome.com.tw/articles/10206169)
    - [從零開始建立自動化發佈的流水線](https://ithelp.ithome.com.tw/articles/10200863)
    - [30天手把手帶你趣學Azure](https://ithelp.ithome.com.tw/articles/10201831)
- Azure DevOps的開發流程:

    ![](https://i.imgur.com/Z5jMe1y.png)

- CICD流程圖


  ![](https://i.imgur.com/8FnKvm1.png)
![](https://i.imgur.com/ffAlRmD.png)


    - **CI (Build Pipelines)**

        
        ![](https://i.imgur.com/ePZZiU4.png)


        - 用.yaml檔來定義CI時要做的事
            - [yaml範例github](https://github.com/microsoft/azure-pipelines-yaml/tree/master/templates)
        - azure-pipelines-example.yml

        ```
        trigger: //在甚麼branch開發時這個yaml檔會被觸發
        	- {{branch}} 
        pool: //要在甚麼image上build這個repoo
          vmImage: 'windows-latest' /

        variables: 
          solution: '**/*.sln'
          buildPlatform: 'Any CPU'
          buildConfiguration: 'Release'
        stages:
        	- stage: Build
        	  displayName: Build stage
        	  jobs:
        	  - job: Build_api
        	    displayName: Build API
        	    steps:
        	    - task: NodeTool@0
        	      inputs:
        	        versionSpec: '10.x'
        	      displayName: 'Install Node.js'
        	    - script: |
        	        npm install
        	      workingDirectory: '$(Build.Repository.LocalPath)/api'
        	      displayName: 'npm install'
        		- job: Build_frontend
        		    displayName: Build Frontend
        		    steps:
        		    - task: NodeTool@0
        		      inputs:
        		        versionSpec: '10.x'
        		      displayName: 'Install Node.js'
        		    - script: |
        		        npm install
        		        npm run build
        		      workingDirectory: '$(Build.Repository.LocalPath)/frontend'
        		      displayName: 'npm install and build'
        	- stage: Test
        	  displayName: Test stage
        	  jobs:
        	  - job: Test_api
        	    displayName: Test API
        	    steps:
        	    - task: SonarCloudPrepare@1
        	      inputs:
        	        SonarCloud: 'SonarCloud training connection'
        	        organization: 'gtrekter'
        	        scannerMode: 'CLI'
        	        configMode: 'manual'
        	        cliProjectKey: 'GTRekter_Training'
        	        cliProjectName: 'Training'
        	        cliSources: '$(Build.Repository.LocalPath)/api'
        	    - task: SonarCloudAnalyze@1
        	    - task: SonarCloudPublish@1
        	      inputs:
        	        pollingTimeoutSec: '300'  
        		- job: Test_frontend
        		    displayName: Test Frontend
        		    steps:
        		    - task: SonarCloudPrepare@1
        		      inputs:
        		        SonarCloud: 'SonarCloud training connection'
        		        organization: 'gtrekter'
        		        scannerMode: 'CLI'
        		        configMode: 'manual'
        		        cliProjectKey: 'GTRekter_Training'
        		        cliProjectName: 'Training'
        		        cliSources: '$(Build.Repository.LocalPath)/frontend'
        		    - task: SonarCloudAnalyze@1
        		    - task: SonarCloudPublish@1
        		      inputs:
        	        pollingTimeoutSec: '300'
        	- stage: Build_dockerimage
        	  displayName: Build and push image to ACR stage
        	  jobs:
        	  - job: Build_api_dockerimage
        	    displayName: Build and push API image
        	    steps:
        	    - task: Docker@2
        	      displayName: Build and push an image to ACR
        	      inputs:
        	        containerRegistry: 'ACR training connection'
        	        repository: 'training-frontend'
        	        command: 'buildAndPush'
        	        Dockerfile: '$(Build.SourcesDirectory)/api/dockerfile'
        	        tags: '$(Build.BuildId)'  
        		- job: Build_frontend_dockerimage
        		    displayName: Build and push frontend image
        		    steps:
        		    - task: Docker@2
        		      displayName: Build and push an image to ACR
        		      inputs:
        		        containerRegistry: 'ACR training connection'
        		        repository: 'training-frontend'
        		        command: 'buildAndPush'
        		        Dockerfile: '$(Build.SourcesDirectory)/frontend/dockerfile'
        		        tags: '$(Build.BuildId)'
        ```

        - 以上example定義好了三個stages(如下)
            - Build
            - Test
            - Build_dockerimage
            - 會出現類似下面的圖(以example來說應該要有三塊build test build_dockerimage)

            
            ![](https://i.imgur.com/wwObzxG.png)


    - **CD (Release Pipelines)**

       ![](https://i.imgur.com/UNLlTRB.png)


        - 剛剛CI若成功後會build一個image出來，而這個image就是CD要用的東西
        - release pipelines中主要分為兩個部分:
            - Artifacts: 要從啥開始release?
                - 要從剛剛CI弄出來的image來deploy，所以add an artifacts要選剛剛的東東
            - Stages: 要經過哪些步驟?
                - SIT、UAT....之類的最後在deploy到真正的環境中

      
        ![](https://i.imgur.com/pNkGTsG.png)

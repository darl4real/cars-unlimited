Service connection required for non Azure Repos can be optionally provided in the command to run it non interatively
az pipelines create --name 'cars-Ulilmited Build' --description 'Pipeline for cars_unlimited project'
--repository https://github.com/darl4real/cars-unlimited --branch master --yml-path azure-pipelines.yml [--service-connection SERVICE_CONNECTION]
- stage: Build
  displayName: Build stage
  jobs: 
- deployment: VMDeploy
  displayName: cars_unlimited
  environment:
    name: <Dev> <QA> <PROD>
    resourceType: VirtualMachine
  strategy:
      rolling:
        maxParallel: 5  #for percentages, mention as x%
        preDeploy:
          steps:
          - download: current
            artifact: drop
          - script: echo initialize, cleanup, backup, install certs
        deploy:
          steps:
          - task: Bash@3
            inputs:
              targetType: 'inline'
              script:  
  - job: Build
    displayName: Build
    - task: NodeTool@0
      inputs:
        versionSpec: '10.x'
      displayName: 'Install Node.js'
    - script: |
        npm install
        npm run build --if-present
        npm run test --if-present
      displayName: 'npm install, build and test'
    - task: ArchiveFiles@2
      displayName: 'Archive files'
      inputs:
        rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
        includeRootFolder: false
        archiveType: zip
        archiveFile: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
        replaceExistingArchive: true
    - upload: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
      artifact: drop
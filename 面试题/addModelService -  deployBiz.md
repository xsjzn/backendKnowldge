```
addModelService -	> deployBiz.deploy  - new DeployStatusEvent(modelServiceId, DeployStatusEnum.DEPLOY_FAILED, e.getMessage())


rebuildModelService 
	1. deployBiz.deploy   - new DeployStatusEvent(modelServiceId, DeployStatusEnum.DEPLOY_FAILED, e.getMessage())
	2. DeployStatusEvent(modelServiceId, removeReferenceVersions)

deleteModelService - > DeployStatusEvent(modelServiceId, removeReferenceVersions)
```
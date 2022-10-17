1. Client submit Hadoop MapReduce Job -> create RM Application (MapReduce AM: MRAppMaster)：
Job.submit() -> JobSubmitter.submitJobInternal() -> YARNRunner.getNewJobID(), YARNRunner.submitJob()

YARNRunner.getNewJobID() -> YarnClientImpl.createApplication() -> ApplicationClientProtocol.getNewApplication()
-> ClientRMService.getNewApplication()

YARNRunner.submitJob() -> YARNRunner.createApplicationSubmissionContext -> set container capability -> YARNRunner.setupLocalResources()
-> YARNRunner.setupAMCommand() -> YARNRunner.setupContainerLaunchContextForAM() -> YARNRunner.setTokenRenewerConf() -> create AM ResourceRequest
-> YarnClientImpl.submitApplication() -> ApplicationClientProtocol.submitApplication() -> ClientRMService.submitApplication()

YARNRunner.submitJob() -> YarnClientImpl.getApplicationReport() -> ApplicationClientProtocol.getApplicationReport()
-> ClientRMService.getApplicationReport()


2. Create Container and trigger it on NM for created application on RM

ClientRMService.submitApplication() -> RMAppManager.submitApplication() -> RMAppManager.createAndPopulateNewRMApp(), RMAppEventType.START

RMAppImpl.RMAppNewlySavingTransition -> RMStateStore.storeNewApplication() -> RMStateStoreEventType.STORE_APP -> RMStateStore.StoreAppTransition -> RMAppEventType.APP_NEW_SAVED
-> RMAppImpl.AddApplicationToSchedulerTransition -> SchedulerEventType.APP_ADDED -> YarnScheduler (FiFoScheduler)
-> FiFoScheduler.addApplication -> Create SchedulerApplication<FifoAppAttempt> -> RMAppEventType.APP_ACCEPTED
-> RMAppImpl.StartAppAttemptTransition -> RMAppImpl.createNewAttempt() -> RMAppAttemptEventType.START

RMAppAttemptImpl.AttemptStartedTransition -> ApplicationMasterService.registerAppAttempt(), SchedulerEventType.APP_ATTEMPT_ADDED
-> FiFoScheduler.addApplicationAttempt -> Create FifoAppAttempt -> RMAppAttemptEventType.ATTEMPT_ADDED
-> RMAppAttemptImpl.ScheduleTransition -> FiFoScheduler.allocate() -> AbstractYarnScheduler.releaseContainers(), Update ContainerUpdates, SchedulerApplicationAttempt.updateResourceRequests
-> AppSchedulingInfo.updateResourceRequests() -> SchedulerApplicationAttempt.pullNewlyAllocatedContainers() (should be empty newly allocated container)

NodeStatusUpdaterImpl.registerWithRM() -> NodeStatusUpdaterImpl.getNMContainerStatuses() -> ResourceTracker.registerNodeManager()
-> ResourceTrackerService.registerNodeManager() -> create RMNodeImpl -> RMNodeEventType.STARTED (or RMNodeEventType.RECONNECTED)

RMNodeImpl.AddNodeTransition -> update running containers and applications in RMNodeImpl based on NMContainerStatuses
-> SchedulerEventType.NODE_ADDED, NodesListManagerEventType.NODE_USABLE
-> FiFoScheduler.addNode(), FiFoScheduler.recoverContainersOnNode() based on NMContainerStatuses
-> ClusterNodeTracker.addNode(), AbstractYarnScheduler.recoverContainersOnNode() based on NMContainerStatuses

NodeStatusUpdaterImpl.startStatusUpdater() -> NodeStatusUpdaterImpl.getNodeStatus() including ContainerStatus by NodeStatusUpdaterImpl.getContainerStatuses()
-> ResourceTracker.nodeHeartbeat() -> ResourceTrackerService.nodeHeartbeat() -> RMNodeImpl.updateNodeHeartbeatResponseForCleanup, 
RMNodeImpl.updateNodeHeartbeatResponseForContainersDecreasing and RMNodeEventType.STATUS_UPDATE

(UNHEALTHY) RMNodeImpl.StatusUpdateWhenUnHealthyTransition -> node health from NM: SchedulerEventType.NODE_ADDED, NodesListManagerEventType.NODE_USABLE

(RUNNING or DECOMMISSIONING) RMNodeImpl.StatusUpdateWhenHealthyTransition -> node not health from NM: RMNodeImpl.reportNodeUnusable()
-> clear RMNodeImpl.nodeUpdateQueue SchedulerEventType.NODE_REMOVED, NodesListManagerEventType.NODE_UNUSABLE

RMNodeImpl.StatusUpdateWhenHealthyTransition -> node health from NM: RMNodeImpl.handleContainerStatus() to get newly launched and completed containers to RMNodeImpl.nodeUpdateQueue,
RMNodeImpl.handleReportedIncreasedContainers() to get newly increased containers to nmReportedIncreasedContainers
-> SchedulerEventType.NODE_UPDATE -> FifoSchedule.nodeUpdate()

FifoSchedule.nodeUpdate() -> AbstractYarnScheduler.nodeUpdate() -> AbstractYarnScheduler.updateNewContainerInfo(): get updates from RMNodeImpl.nodeUpdateQueue
call AbstractYarnScheduler.containerLaunchedOnNode() -> SchedulerApplicationAttempt.containerLaunchedOnNode() for newly launched containers;
get newly increased containers from nmReportedIncreasedContainers, call AbstractYarnScheduler.containerIncreasedOnNode() for newly increased containers
-> RMContainerEventType.LAUNCHED for newly launched containers, RMContainerEventType.NM_DONE_CHANGE_RESOURCE for newly increased containers
-> AbstractYarnScheduler.updateCompletedContainers() -> AbstractYarnScheduler.completedContainer(), AbstractYarnScheduler.releaseContainer()

FifoSchedule.nodeUpdate() -> node available resource > minimumAllocation then FifoSchedule.assignContainers()
-> FifoSchedule.getMaxAllocatableContainers() and FifoSchedule.assignContainersOnNode() -> FifoSchedule.assignContainer()
-> "FiFoAppAttempt.allocate() -> new RMContainerImpl, SchedulerApplicationAttempt.addToNewlyAllocatedContainers, AppSchedulingInfo.allocate() and RMContainerEventType.START -> addToNewlyAllocatedContainers"
and SchedulerNode.allocateContainer()

RMContainerEventType.START -> RMContainerImpl.ContainerStartedTransition -> RMAppAttemptEventType.CONTAINER_ALLOCATED -> RMAppAttemptImpl.AMContainerAllocatedTransition
-> FiFoScheduler.allocate() -> SchedulerApplicationAttempt.pullNewlyAllocatedContainers() (get newly allocated AM container) -> SchedulerApplicationAttempt.updateContainerAndNMToken()
-> RMContainerEventType.ACQUIRED/RMContainerEventType.ACQUIRE_UPDATED_CONTAINER -> RMContainerImpl.AcquiredTransition/RMContainerImpl.ContainerAcquiredWhileRunningTransition
-> register allocation timeout and RMAppEventType.APP_RUNNING_ON_NODE/register allocation timeout if increasing container
-> RMAppAttemptImple.storeAttempt() -> RMStateStore.storeNewApplicationAttempt() -> RMStateStoreEventType.STORE_APP_ATTEMPT -> RMStateStore.StoreAppAttemptTransition
-> RMAppAttemptEventType.ATTEMPT_NEW_SAVED -> RMAppAttemptImpl.AttemptStoredTransition -> RMAppAttemptImpl.launchAttempt()
-> AMLauncherEventType.LAUNCH -> ApplicationMasterLauncher.launch() -> create AMLauncher

ApplicationMasterLauncher.run() -> AMLauncher.run() -> AMLauncher.launch() -> ContainerManagementProtocol.startContainers() -> ContainerManagerImpl.startContainers()
AMLauncher.run() -> RMAppAttemptEventType.LAUNCHED -> RMAppAttemptImpl.AMLaunchedTransition

When container launched on NM to run AM application -> ApplicationMasterService.registerApplicationMaster() -> RMAppAttemptEventType.REGISTERED
-> RMAppAttemptImpl.AMRegisteredTransition -> RMAppEventType.ATTEMPT_REGISTERED -> RMAppImpl.RMAppStateUpdateTransition

3. Launch container to run AM for the application on NM

ContainerManagerImpl.startContainers() -> ContainerManagerImpl.startContainerInternal() -> create ContainerImpl and ApplicationImpl on NM 
-> ApplicationEventType.INIT_APPLICATION, ApplicationEventType.INIT_CONTAINER -> ApplicationImpl.AppInitTransition, ApplicationImpl.InitContainerTransition 
-> LogHandlerEventType.APPLICATION_STARTED -> LogAggregationService.initApp() -> ApplicationEventType.APPLICATION_LOG_HANDLING_INITED/ApplicationEventType.APPLICATION_LOG_HANDLING_FAILED
-> ApplicationImpl.AppLogInitDoneTransition/ApplicationImpl.AppLogInitFailTransition -> LocalizationEventType.INIT_APPLICATION_RESOURCES
-> ResourceLocalizationService.handleInitApplicationResources -> ApplicationEventType.APPLICATION_INITED -> Application.AppInitDoneTransition
-> ContainerEventType.INIT_CONTAINER -> ContainerImpl.RequestResourcesTransition -> (AuxServicesEventType.CONTAINER_INIT, AuxServicesEventType.APPLICATION_INIT)
-> LocalizationEventType.LOCALIZE_CONTAINER_RESOURCES -> ResourceLocalizationService.handleInitContainerResources -> ResourceEventType.REQUEST -> LocalizedResource.FetchResourceTransition
-> LocalizerEventType.REQUEST_RESOURCE_LOCALIZATIO -> ResourceLocalizationService.LocalizerTracker.handle() -> LocalResourcesTrackerImpl.handle() -> ResourceEventType.LOCALIZED
-> LocalizedResource.FetchSuccessTransition -> ContainerEventType.RESOURCE_LOCALIZED -> ContainerImpl.LocalizedTransition 
-> LocalizationEventType.CONTAINER_RESOURCES_LOCALIZED -> ResourceLocalizationService.handleContainerResourcesLocalized 
-> LocalizerTracker.endContainerLocalization() -> ContainerImpl.sendScheduleEvent() -> ContainerSchedulerEventType.SCHEDULE_CONTAINER 
-> ContainerScheduler.scheduleContainer() -> ContainerScheduler.startAllocatedContainer() -> ContainerImpl.sendLaunchEvent() 
-> ContainersLauncherEventType.LAUNCH_CONTAINER -> ContainersLauncher.handle() -> new ContainerLaunch -> ContainerLaunch.call() -> ContainerLaunch.expandEnvironment() replace placeholders in commands and environment values
-> ContainerLaunch.sanitizeEnv() -> ContainerExecutor.writeLaunchEnv() write down the application script running in container 
-> ContainerLaunch.launchContainer() -> ContainerEventType.CONTAINER_LAUNCHED -> ContainerImpl.LaunchTransition, ContainerExecutor.activateContainer()
-> LinuxContainerExecutor.launchContainer() -> ResourceHandlerChain.preStart() prepare resource limits via CGgroup -> PrivilegedOperation.OperationType.ADD_PID_TO_CGROUP
-> LinuxContainerExecutor.addSchedPriorityCommand() for set priority -> ContainerRuntimeContext.Builder.build() build start command to call contain-execute for running the application script in container
-> LinuxContainerRuntime.launchContainer()
              
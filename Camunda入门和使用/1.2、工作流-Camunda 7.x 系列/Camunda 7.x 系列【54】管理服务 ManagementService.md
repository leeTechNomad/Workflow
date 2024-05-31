> **有道无术，术尚可求，有术无道，止于术。**
> 
> **本系列Spring Boot 版本 2.7.9**
> 
> **本系列Camunda 版本 7.19.0**
> 
> **源码地址：https://gitee.com/pearl-organization/camunda-study-demo**

### 1\. 概述

`ManagementService`管理服务，负责[流程引擎](https://so.csdn.net/so/search?q=%E6%B5%81%E7%A8%8B%E5%BC%95%E6%93%8E&spm=1001.2101.3001.7020)的管理和维护操作，允许用户获取关于数据库表及其元数据的信息，以及关于作业的查询能力和管理操作。

### 2\. Java API

#### 2.1 数据库元数据

        /**
         * 获取包含数据库架构的｛表名，行计数｝项的映射。
         */
        Map<String, Long> getTableCount();
    
        /**
         * 获取像｛@link Task｝、｛@link Execution｝等实体的表名（包括任何配置的前缀）。
         */
        String getTableName(Class<?> entityClass);
    
        /**
         * 获取某个表的元数据（列名、列类型等）。当不存在具有给定名称的表时，返回null。
         */
        TableMetaData getTableMetaData(String tableName);
    
        /**
         * 创建一个｛@link TablePageQuery｝，该查询可用于提取包含表行数据的特定部分的｛@linkTablePage｝。
         */
        TablePageQuery createTablePageQuery();
    

#### 2.2 查询Job和Job定义

        /**
         * 返回一个新的JobQuery实现，该实现可用于动态查询作业。
         */
        JobQuery createJobQuery();
    
        /**
         * 返回一个新的｛@link JobDefinitionQuery｝实现，该实现可用于动态查询作业定义。
         */
        JobDefinitionQuery createJobDefinitionQuery();
    

#### 2.3 激活 & 挂起Job定义

     /**
         * 立即激活作业定义
         */
        void activateJobDefinitionById(String jobDefinitionId);
        void activateJobDefinitionByProcessDefinitionId(String processDefinitionId);
        void activateJobDefinitionByProcessDefinitionKey(String processDefinitionKey);
        void activateJobDefinitionById(String jobDefinitionId, boolean activateJobs);
        void activateJobDefinitionByProcessDefinitionId(String processDefinitionId, boolean activateJobs);
        void activateJobDefinitionByProcessDefinitionKey(String processDefinitionKey, boolean activateJobs);
        void activateJobDefinitionById(String jobDefinitionId, boolean activateJobs, Date activationDate);
        void activateJobDefinitionByProcessDefinitionId(String processDefinitionId, boolean activateJobs, Date activationDate);
        void activateJobDefinitionByProcessDefinitionKey(String processDefinitionKey, boolean activateJobs, Date activationDate);
    
        /**
         * 立即挂起作业定义
         */
        void suspendJobDefinitionById(String jobDefinitionId);
        void suspendJobDefinitionByProcessDefinitionId(String processDefinitionId);
        void suspendJobDefinitionByProcessDefinitionKey(String processDefinitionKey);
        void suspendJobDefinitionById(String jobDefinitionId, boolean suspendJobs);
        void suspendJobDefinitionByProcessDefinitionId(String processDefinitionId, boolean suspendJobs);
        void suspendJobDefinitionByProcessDefinitionKey(String processDefinitionKey, boolean suspendJobs);
        void suspendJobDefinitionById(String jobDefinitionId, boolean suspendJobs, Date suspensionDate);
        void suspendJobDefinitionByProcessDefinitionId(String processDefinitionId, boolean suspendJobs, Date suspensionDate);
        void suspendJobDefinitionByProcessDefinitionKey(String processDefinitionKey, boolean suspendJobs, Date suspensionDate);
        // 基于构建者修改作业定义状态
        UpdateJobDefinitionSuspensionStateSelectBuilder updateJobDefinitionSuspensionState();
    

#### 2.4 激活 & 挂起Job

        /**
         * 激活作业
         */
        void activateJobById(String jobId);
        void activateJobByJobDefinitionId(String jobDefinitionId);
        void activateJobByProcessInstanceId(String processInstanceId);
        void activateJobByProcessDefinitionId(String processDefinitionId);
        void activateJobByProcessDefinitionKey(String processDefinitionKey);
    
        /**
         * 挂起作业
         */
        void suspendJobById(String jobId);
        void suspendJobByJobDefinitionId(String jobDefinitionId);
        void suspendJobByProcessInstanceId(String processInstanceId);
        void suspendJobByProcessDefinitionId(String processDefinitionId);
        void suspendJobByProcessDefinitionKey(String processDefinitionKey);
        // 基于构建者修改作业状态
        UpdateJobSuspensionStateSelectBuilder updateJobSuspensionState();
    

#### 2.5 重试次数

        /**
         * 设置作业剩余的重试次数。每当JobExecutor无法执行作业时，该值就会递减。
         * 当它达到零时，该作业应该已终止，并且不会再次重试。在这种情况下，可以使用此方法来增加重试次数。
         *
         * @param jobId   要修改的作业的id不能为null。
         * @param retries 重试次数。
         */
        void setJobRetries(String jobId, int retries);
        void setJobRetries(List<String> jobIds, int retries);
        SetJobRetriesBuilder setJobRetries(int retries);
        Batch setJobRetriesAsync(List<String> jobIds, int retries);
        Batch setJobRetriesAsync(JobQuery jobQuery, int retries);
        Batch setJobRetriesAsync(List<String> jobIds, JobQuery jobQuery, int retries);
        Batch setJobRetriesAsync(List<String> processInstanceIds, ProcessInstanceQuery query, int retries);
        Batch setJobRetriesAsync(List<String> processInstanceIds,
                                 ProcessInstanceQuery processInstanceQuery,
                                 HistoricProcessInstanceQuery historicProcessInstanceQuery,
                                 int retries);
        SetJobRetriesByJobsAsyncBuilder setJobRetriesByJobsAsync(int retries);
        SetJobRetriesByProcessAsyncBuilder setJobRetriesByProcessAsync(int retries);
    void setJobRetriesByJobDefinitionId(String jobDefinitionId, int retries);
    

#### 2.6 到期时间

        /**
         * 设置作业的到期日期
         */
        void setJobDuedate(String jobId, Date newDuedate);
        void setJobDuedate(String jobId, Date newDuedate, boolean cascade);
        /**
         * 触发作业的重新计算
         * @param jobId id既不能为null也不能为空
         * @param creationDateBased 指示重新计算应基于作业的创建日期还是当前日期
         */
        void recalculateJobDuedate(String jobId, boolean creationDateBased);
    

#### 2.7 优先级

        /**
         * 为给定作业显式设置优先级，此设置将覆盖BPMN 2.0 XML中指定的任何设置
         *
         * @param jobDefinitionId 作业定义ID
         * @param priority        要设置的优先级
         * @param cascade         如果为true，定义的的所有作业的优先级也会更改
         */
        void setOverridingJobPriorityForJobDefinition(String jobDefinitionId, long priority, boolean cascade);
        void setOverridingJobPriorityForJobDefinition(String jobDefinitionId, long priority);
        /**
         * 如果之前设置了覆盖优先级，则会清除。接收BPMN2.0XML中指定的优先级或全局默认优先级， 现有作业实例优先级保持不变
         */
        void clearOverridingJobPriorityForJobDefinition(String jobDefinitionId);
    

#### 2.8 ProcessApplication

        /**
         * 激活给定ProcessApplication的部署。此方法的效果有两个：
         * 1. 流程引擎将在该ProcessApplication的上下文中执行原子操作
         * 2. 作业执行器将开始从该部署获取作业
         */
        ProcessApplicationRegistration registerProcessApplication(String deploymentId, ProcessApplicationReference reference);
    
        /**
         * 停用给定ProcessApplication的部署。这将删除流程引擎和流程应用程序之间的关联，并可选地从缓存中删除关联的流程定义。
         *
         * @param deploymentId                      要停用的部署的Id
         * @param removeProcessDefinitionsFromCache 指示是否应从部署缓存中删除流程定义
         */
        void unregisterProcessApplication(String deploymentId, boolean removeProcessDefinitionsFromCache);
    
        void unregisterProcessApplication(Set<String> deploymentIds, boolean removeProcessDefinitionsFromCache);
        /**
         * 当前为给定部署注册的流程应用程序的名称，如果当前未注册流程应用程序，则为“null”。
         */
        String getProcessApplicationForDeployment(String deploymentId);
    

#### 2.9 属性和许可证

        /**
         * 获取所有属性的映射。
         */
        Map<String, String> getProperties();
    
        /**
         * 设置属性的值
         * @param name  属性的名称
         * @param value 属性的新值。
         */
        void setProperty(String name, String value);
    
        /**
         * 按名称删除特性。如果该属性不存在，则会忽略该请求@param name要删除的属性的名称
         */
        void deleteProperty(String name);
        
        /**
         * 设置许可证密钥
         * @param licenseKey 许可证密钥字符串。
         */
        void setLicenseKey(String licenseKey);
    
        /**
         * 如果未设置许可证，则获取存储的许可证密钥字符串或＜code＞null＜code＞。
         */
        String getLicenseKey();
    
        /**
         * 删除存储的许可证密钥。如果未设置许可证密钥，则会忽略该请求。
         */
        void deleteLicenseKey();
    

#### 2.10 度量

        /**
         * 返回一个新的度量查询
         */
        MetricsQuery createMetricsQuery();
    
        /**
         * 删除所有早于指定时间戳的度量事件。如果时间戳为空，则将删除所有度量
         */
        void deleteMetrics(Date timestamp);
    
        /**
         * 删除所有早于指定时间戳并由给定报告程序报告的度量事件。如果某个参数为null，则所有度量事件在这方面都匹配。
         *
         * @param timestamp or null
         * @param reporter  or null
         */
        void deleteMetrics(Date timestamp, String reporter);
    
        /**
         * 强制此引擎将其挂起的收集度量提交到数据库。
         */
        void reportDbMetricsNow();
    
        /**
         * 删除所有早于指定时间戳的任务度量。如果时间戳为空，则将删除所有度量
         */
        void deleteTaskMetrics(Date timestamp);
    

#### 2.11 批处理

        /**
         * 创建查询以搜索｛@link Batch｝实例。
         */
        BatchQuery createBatchQuery();
    
        /**
         * 立即挂起具有给定id的批处理，相关的所有｛@link JobDefinition｝和｛@linkJob｝都将被挂起
         */
        void suspendBatchById(String batchId);
    
        /**
         * 立即激活具有给定id的批处理，相关的所有｛@link JobDefinition｝和｛@linkJob｝都将被激活
         */
        void activateBatchById(String batchId);
    
        /**
         * 删除批处理实例和相应的作业定义。如果级联设置为true，则历史批处理实例和历史作业日志也将被删除。
         */
        void deleteBatch(String batchId, boolean cascade);
    
        /**
         * 查询一个批次的批次执行作业的统计信息。
         */
        BatchStatisticsQuery createBatchStatisticsQuery();
    

#### 2.12 遥测数据

        /**
         * 查询数据库架构日志的条目。
         */
        SchemaLogQuery createSchemaLogQuery();
    
        /**
         * 启用禁用向Camunda发送遥测数据
         */
        void toggleTelemetry(boolean enabled);
    
        /**
         * 检查如何配置向Camunda发送遥测数据
         */
        Boolean isTelemetryEnabled();
    
        /**
         * 此方法返回收集的遥测数据的当前状态。
         */
        TelemetryData getTelemetryData();
    

#### 2.13 执行 & 删除作业

        /**
         * 强制同步执行作业（例如，用于管理或测试）即使流程定义和或流程实例处于挂起状态，也会执行作业。
         *
         * @param jobId 要执行的作业的id不能为null。
         */
        void executeJob(String jobId);
    
        /**
         * 删除具有提供的id的作业。
         */
        void deleteJob(String jobId);
    
    

#### 2.14 其他

        /**
         * 返回上次执行具有给定id的作业时发生的异常的完整堆栈跟踪。当作业没有异常stacktrace时，返回null@作业的param jobId id，不能为null。
         */
        String getJobExceptionStacktrace(String jobId);
    
        /**
         * 给定连接上的程序化架构更新返回有关所发生情况的反馈注意：将始终返回空字符串
         */
        String databaseSchemaUpgrade(Connection connection, String catalog, String schema);
    
        /**
         * 查询按流程定义聚合的流程实例数。
         */
        ProcessDefinitionStatisticsQuery createProcessDefinitionStatisticsQuery();
    
        /**
         * 查询按部署聚合的流程实例数。
         */
        DeploymentStatisticsQuery createDeploymentStatisticsQuery();
    
        /**
         * 查询单个流程定义的活动聚合的活动实例数。
         */
        ActivityStatisticsQuery createActivityStatisticsQuery(String processDefinitionId);
    
        /**
         * 获取已注册引擎作业执行器的部署。仅当设置了引擎配置属性＜code＞jobExecutiorDeploymentAware＜code＞时，此设置才相关。
         */
        Set<String> getRegisteredDeployments();
    
        /**
         * 为引擎的作业执行器注册部署。如果设置了引擎配置属性jobExecutiorDeploymentAware，则这是必需的。如果设置为false，则作业执行器将执行任何作业。
         */
        void registerDeploymentForJobExecutor(String deploymentId);
    
        /**
         * 注销引擎作业执行器的部署。如果设置了引擎配置属性jobExecutionDeploymentAware，则将不再获取给定部署的作业。
         */
        void unregisterDeploymentForJobExecutor(String deploymentId);
    
        /**
         * 获取流程引擎的配置历史记录级别@返回历史级别
         */
        int getHistoryLevel();
    
        /**
         * 基于用户任务分配计算唯一任务工作者的数量
         * @param startTime 限制在给定日期（含）之后收集的数据，
         * @param endTime 限制在指定日期之前收集的数据（独占），
         * @ 返回唯一任务工作者的总数（可以限制在某个时间间隔内）
         */
        long getUniqueTaskWorkerCount(Date startTime, Date endTime);
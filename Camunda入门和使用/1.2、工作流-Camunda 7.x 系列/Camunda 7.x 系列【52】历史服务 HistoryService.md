> **有道无术，术尚可求，有术无道，止于术。**
> 
> **本系列Spring Boot 版本 2.7.9**
> 
> **本系列Camunda 版本 7.19.0**
> 
> **源码地址：https://gitee.com/pearl-organization/camunda-study-demo**

### 1\. 概述

`HistoryService`历史服务提供了[历史数据](https://so.csdn.net/so/search?q=%E5%8E%86%E5%8F%B2%E6%95%B0%E6%8D%AE&spm=1001.2101.3001.7020)相关操作的`Java API`。

`RuntimeService`用于管理流程运行时相关[动态数据](https://so.csdn.net/so/search?q=%E5%8A%A8%E6%80%81%E6%95%B0%E6%8D%AE&spm=1001.2101.3001.7020)，当运行结束后，对应`act_ru_xx`开头表中的数据都会被删除，历史数据会保存在`act_hi_xx`开头的表中，进行独立永久性存储，并经过优化处理利于查询。

**注意：** 流程运行时，部分运行时数据也会同时保存到历史库中，所以`HistoryService`可以查询到正在运行运行以及完成结束的相关数据。

### 2\. Java API

#### 2.1 流程实例

通过构建条件查询对象，查询流程实例历史数据。

     HistoricProcessInstanceQuery createHistoricProcessInstanceQuery();
    

示例代码：

        @Test
        @DisplayName("查询流程实例历史数据")
        void createHistoricProcessInstanceQuery() {
            // 构建查询对象，并执行列表查询
            //
            List<HistoricProcessInstance> instanceList = historyService.createHistoricProcessInstanceQuery()
                    //.processInstanceBusinessKey(businessKey) // 业务KEY
                    //.processInstanceId(processInstanceId) // 流程实例ID
                    .finished() // 已完成的（默认也会查询正在运行的实例）
                    .startedBy("admin") // 流程发起人
                    .list();
            for (HistoricProcessInstance historicProcessInstance : instanceList) {
                System.out.println("ID："+historicProcessInstance.getId());
                System.out.println("开始时间："+historicProcessInstance.getStartTime());
                System.out.println("结束时间："+historicProcessInstance.getEndTime());
            }
        }
    

#### 2.2 活动

通过构建条件查询对象，查询活动节点历史数据。

    HistoricActivityInstanceQuery createHistoricActivityInstanceQuery();
    // 根据流程定义`ID`查询聚合的历史活动实例数
    HistoricActivityStatisticsQuery createHistoricActivityStatisticsQuery(String processDefinitionId);
    

示例代码：

        @Test
        @DisplayName("查询历史活动节点")
        void createHistoricActivityInstanceQuery() {
            // 已完成
            String processInstanceId = "3347ae41-4b95-11ee-a21b-00ff045b1bb6";
            List<HistoricActivityInstance> historicActivities = historyService.createHistoricActivityInstanceQuery()
                    .processInstanceId(processInstanceId) // 流程实例ID
                    .orderByHistoricActivityInstanceEndTime().asc() // 完成时间升序
                    .finished() // 查询已完成的活动实例
                    .list();
            for (HistoricActivityInstance historicActivity : historicActivities) {
                System.out.println("活动节点ID：" + historicActivity.getActivityId());
                System.out.println("活动节点名称：" + historicActivity.getActivityName());
            }
        }
    

#### 2.3 任务

通过构建条件查询对象，查询任务历史数据。

    HistoricTaskInstanceQuery createHistoricTaskInstanceQuery() 
    

示例代码：

        @Test
        @DisplayName("查询历史任务")
        void createHistoricTaskInstanceQuery() {
            String processInstanceId = "3347ae41-4b95-11ee-a21b-00ff045b1bb6";
            List<HistoricTaskInstance> taskInstanceList = historyService.createHistoricTaskInstanceQuery()
                    .processInstanceId(processInstanceId) // 流程实例ID
                    .finished()  // 已完成的
                    .list();
        }
    

#### 2.4 变量

可以查询历史变量。

        /**
         * 构建 HistoricDetail 查询对象，用于查询变量详细
         */
        HistoricDetailQuery createHistoricDetailQuery();
    
        /**
         * 查询流程变量，包含其流程实例完成时的最后一个值。仅当HISTORY_LEVEL设置为 >= AUDIT时才可用
         */
        HistoricVariableInstanceQuery createHistoricVariableInstanceQuery();
    

示例代码：

        @Test
        @DisplayName("查询历史变量")
        void createHistoricDetailQuery() {
            // 查询历史变量详情
            String processInstanceId = "3347ae41-4b95-11ee-a21b-00ff045b1bb6";
            List<HistoricDetail> details = historyService.createHistoricDetailQuery()
                    .processInstanceId(processInstanceId) // 流程实例ID
                    .list();
            // 历史变量
            List<HistoricVariableInstance> list = historyService.createHistoricVariableInstanceQuery()
                    .processInstanceId(processInstanceId)
                    .list();
        }
    

#### 2.5 删除

历史数据可以进行删除。

        /**
         * 删除历史任务实例。如果历史任务实例不存在，则不会引发异常，并且方法返回正常。
         */
        void deleteHistoricTaskInstance(String taskId);
    
        /**
         * 删除历史进程实例。所有历史活动、历史任务和历史细节（变量更新、表单属性）也将被删除。
         */
        void deleteHistoricProcessInstance(String processInstanceId);
    
        /**
         * 删除历史进程实例。所有历史活动、历史任务和历史细节（变量更新、表单属性）也将被删除。如果未找到流程实例，则不会失败。
         */
        void deleteHistoricProcessInstanceIfExists(String processInstanceId);
    
        /**
         * 删除历史进程实例。所有历史活动、历史任务和历史细节（变量更新、表单属性）也将被删除。
         */
        void deleteHistoricProcessInstances(List<String> processInstanceIds);
    
        /**
         * 删除历史进程实例。所有历史活动、历史任务和历史细节（变量更新、表单属性）也将被删除。如果未找到流程实例，则不会失败。
         */
        void deleteHistoricProcessInstancesIfExists(List<String> processInstanceIds);
    
        /**
         * 批量删除历史进程实例和所有相关的历史数据。将为每个实体类型创建DELETE SQL语句。他们将在in子句中有给定进程实例ID的列表。因此，必须考虑DB对in子句中值数量的限制。
         */
        void deleteHistoricProcessInstancesBulk(List<String> processInstanceIds);
        
        /**
         * 按id删除历史变量实例。所有相关的历史详细信息（变量更新、表单属性）也将被删除。
         */
        void deleteHistoricVariableInstance(String variableInstanceId);
    
        /**
         * 删除流程实例的所有历史变量和历史详细信息（变量更新、表单属性）。
         */
        void deleteHistoricVariableInstancesByProcessInstanceId(String processInstanceId);
        
        /**
         * 异步删除历史进程实例。所有历史活动、历史任务和历史细节（变量更新、表单属性）也将被删除。
         */
        Batch deleteHistoricProcessInstancesAsync(List<String> processInstanceIds, String deleteReason);
    
        /**
         * 基于查询异步删除历史进程实例。所有历史活动、历史任务和历史细节（变量更新、表单属性）也将被删除。
         */
        Batch deleteHistoricProcessInstancesAsync(HistoricProcessInstanceQuery query, String deleteReason);
    
        /**
         * 异步删除历史流程实例。查询结果和ID列表将合并。所有历史活动、历史任务和历史细节（变量更新、表单属性）也将被删除。
         */
        Batch deleteHistoricProcessInstancesAsync(List<String> processInstanceIds, HistoricProcessInstanceQuery query, String deleteReason);
    
        /**
         * 删除用户操作日志项。不级联到任何相关实体。
         */
        void deleteUserOperationLogEntry(String entryId);
    
        /**
         * 删除历史批处理实例，所有相应的历史作业日志也被删除。
         */
        void deleteHistoricBatch(String id);
    

#### 2.6 原生查询

提供流程实例、任务、活动、变量等历史数据，提供了基于`SQL`查询的方法。

        NativeHistoricProcessInstanceQuery createNativeHistoricProcessInstanceQuery();
        NativeHistoricTaskInstanceQuery createNativeHistoricTaskInstanceQuery();
        NativeHistoricActivityInstanceQuery createNativeHistoricActivityInstanceQuery();
        NativeHistoricCaseInstanceQuery createNativeHistoricCaseInstanceQuery();
        NativeHistoricCaseActivityInstanceQuery createNativeHistoricCaseActivityInstanceQuery();
        NativeHistoricDecisionInstanceQuery createNativeHistoricDecisionInstanceQuery();
        NativeHistoricVariableInstanceQuery createNativeHistoricVariableInstanceQuery();
    

#### 2.7 清理

为了防止历史数据过多，导致数据库性能下降，支持设置过期时间，流程引擎会开启作业去执行清理操作，比如下面表示，该流程定义历史数据保存时间为`90`天， 也可以直接调用`API`清理已经过期的历史数据。

![在这里插入图片描述](data:image/png;base64,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)

        /**
         * 异步批处理清理历史数据
         */
        Job cleanUpHistoryAsync();
    
        /**
         * 立即异步批处理清理历史数据
         */
        Job cleanUpHistoryAsync(boolean immediatelyDue);
    
        /**
         * 查找历史清理作业（如果存在）。
         */
        List<Job> findHistoryCleanupJobs();
        
        /**
         * 为历史流程实例和所有关联的历史实体设置删除时间。
         */
        SetRemovalTimeSelectModeForHistoricProcessInstancesBuilder setRemovalTimeToHistoricProcessInstances();
    
        /**
         * 为历史批次和所有关联的历史实体设置删除时间
         */
        SetRemovalTimeSelectModeForHistoricBatchesBuilder setRemovalTimeToHistoricBatches();
    

#### 2.8 其他

        /**
         * 查询用户操作日志历史数据
         */
        UserOperationLogQuery createUserOperationLogQuery();
    
        /**
         * 查询突发事件历史数据
         */
        HistoricIncidentQuery createHistoricIncidentQuery();
    
        /**
         * 查询参与者关联历史数据
         */
        HistoricIdentityLinkLogQuery createHistoricIdentityLinkLogQuery();
    
        /**
         * 查询作业执行日志
         */
        HistoricJobLogQuery createHistoricJobLogQuery();
    
        
    
        /**
         * 查询历史作业日志时发生的异常的完整堆栈跟踪，当历史作业日志没有异常时，返回null。
         */
        String getHistoricJobLogExceptionStacktrace(String historicJobLogId);
    
        /**
         * 查询以创建历史流程实例报告。
         */
        HistoricProcessInstanceReport createHistoricProcessInstanceReport();
    
        /**
         * 查询以创建历史任务实例报告。
         */
        HistoricTaskInstanceReport createHistoricTaskInstanceReport();
    
        /**
         * 查询可清理的历史流程实例报告。
         */
        CleanableHistoricProcessInstanceReport createCleanableHistoricProcessInstanceReport();
    
        /**
         * 查询可清理的历史批处理报告。
         *
         */
        CleanableHistoricBatchReport createCleanableHistoricBatchReport();
    
        /**
         * 查询历史批处理
         */
        HistoricBatchQuery createHistoricBatchQuery();
    
        /**
         * 查询DRD评估的统计信息。
         */
        HistoricDecisionInstanceStatisticsQuery createHistoricDecisionInstanceStatisticsQuery(String decisionRequirementsDefinitionId);
    
        /**
         * 查询外部任务日志历史记录
         */
        HistoricExternalTaskLogQuery createHistoricExternalTaskLogQuery();
    
        /**
         * 获取历史外部任务日志错误详细信息
         */
        String getHistoricExternalTaskLogErrorDetails(String historicExternalTaskLogId);
        
    
        /**
         * 为用户操作日志设置注释
         */
        void setAnnotationForOperationLogById(String operationId, String annotation);
    
        /**
         * 清除用户操作日志的注释
         */
        void clearAnnotationForOperationLogById(String operationId);
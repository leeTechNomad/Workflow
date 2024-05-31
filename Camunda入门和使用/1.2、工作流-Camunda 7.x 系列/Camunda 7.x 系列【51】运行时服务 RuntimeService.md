> **有道无术，术尚可求，有术无道，止于术。**
> 
> **本系列Spring Boot 版本 2.7.9**
> 
> **本系列Camunda 版本 7.19.0**
> 
> **源码地址：https://gitee.com/pearl-organization/camunda-study-demo**

### 1\. 概述

`RuntimeService`运行时服务，提供了对执行对象、流程实例、流程变量等[动态数据](https://so.csdn.net/so/search?q=%E5%8A%A8%E6%80%81%E6%95%B0%E6%8D%AE&spm=1001.2101.3001.7020)的管理，是核心组件之一。本篇文档包含了`RuntimeService`中所有的方法`API`。

### 2\. API

#### 2.1 ID、Key 启动

可以使用`RuntimeService`通过流程定义的`ID`或`KEY`启动一个流程实例。

        /**
         * 启动流程实例（使用最新版本的流程定义）
         *
         * @param processDefinitionKey 进程定义的KEY键，不能为null
         * @throws ProcessEngineException 没有进程定义异常
         * @throws AuthorizationException 权限异常
         */
        ProcessInstance startProcessInstanceByKey(String processDefinitionKey);
    
        /**
         * 启动流程实例（使用最新版本的流程定义）
         *
         * @param processDefinitionKey 进程定义的KEY键
         * @param businessKey          在上下文中唯一标识流程实例的键
         */
        ProcessInstance startProcessInstanceByKey(String processDefinitionKey, String businessKey);
    
        /**
         * 启动流程实例（使用最新版本的流程定义）
         *
         * @param processDefinitionKey 进程定义的KEY键，不能为null
         * @param variables            要传递的变量，可以为null
         */
        ProcessInstance startProcessInstanceByKey(String processDefinitionKey, Map<String, Object> variables);
    
        /**
         * 启动流程实例（使用最新版本的流程定义）
         *
         * @param processDefinitionKey 进程定义的KEY键，不能为null
         * @param variables            要传递的变量，可以为null
         * @param businessKey          业务Key
         */
        ProcessInstance startProcessInstanceByKey(String processDefinitionKey, String businessKey, Map<String, Object> variables);
    
        /**
         * 启动流程实例（使用流程定义ID）
         *
         * @param processDefinitionId 流程定义ID，不能为null
         */
        ProcessInstance startProcessInstanceById(String processDefinitionId);
    
        /**
         * 启动流程实例（使用流程定义ID）
         *
         * @param processDefinitionId 流程定义ID，不能为null
         * @param businessKey         业务Key
         */
        ProcessInstance startProcessInstanceById(String processDefinitionId, String businessKey);
    
    
        /**
         * 启动流程实例（使用流程定义ID）
         *
         * @param processDefinitionId 流程定义ID，不能为null
         * @param variables           要传递的变量，可以为null
         */
        ProcessInstance startProcessInstanceById(String processDefinitionId, Map<String, Object> variables);
    
        /**
         * 启动流程实例（使用流程定义ID）
         *
         * @param processDefinitionId 流程定义ID，不能为null
         * @param variables           要传递的变量，可以为null
         * @param businessKey         业务Key
         */
        ProcessInstance startProcessInstanceById(String processDefinitionId, String businessKey, Map<String, Object> variables);
    

[示例代码](https://so.csdn.net/so/search?q=%E7%A4%BA%E4%BE%8B%E4%BB%A3%E7%A0%81&spm=1001.2101.3001.7020)：

    @RequestMapping("/runtime")
    @RestController
    @Tag(name = "运行服务")
    public class RuntimeController {
    
        @Autowired
        private RuntimeService runtimeService;
        
        @PostMapping("/startProcessInstanceByKey")
        @Operation(summary = "通过流程定义KEY启动流程")
        public R<String> startProcessInstanceByKey(String processDefinitionId, String businessKey, String caseInstanceId, Map<String, Object> variables) {
            ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processDefinitionId, businessKey, caseInstanceId, variables);
            return R.success(processInstance.getProcessInstanceId());
        }
        
        @PostMapping("/startProcessInstanceById")
        @Operation(summary = "通过流程定义ID启动流程")
        public R<String> startProcessInstanceById(String processDefinitionId, String businessKey, String caseInstanceId, Map<String, Object> variables) {
            ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefinitionId, businessKey, caseInstanceId, variables);
            return R.success(processInstance.getProcessInstanceId());
        }
    }
    

在实际应用中，更推荐使用构建者的方式启动新的流程实例：

        /**
         * 通过构建者的方式启动新的流程实例（推荐使用）
         */
        ProcessInstantiationBuilder createProcessInstanceById(String processDefinitionId);
        ProcessInstantiationBuilder createProcessInstanceByKey(String processDefinitionKey);
    

#### 2.2 消息启动

对于需要消息事件启动的流程，`RuntimeService`可以发送特定消息，如果消息能被相应的消息启动事件捕获，则会启动一个新的流程实例，如果不存在对应的消息订阅者，则会抛出`ProcessEngineException`。

        /**
         * 向流程引擎发出消息，并启动新的流程实例
         *
         * @param messageName 消息的“名称”
         * @since 5.9
         */
        ProcessInstance startProcessInstanceByMessage(String messageName);
        ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey);
        ProcessInstance startProcessInstanceByMessage(String messageName, Map<String, Object> processVariables);
        ProcessInstance startProcessInstanceByMessage(String messageName, String businessKey, Map<String, Object> processVariables);
        ProcessInstance startProcessInstanceByMessageAndProcessDefinitionId(String messageName, String processDefinitionId);
        ProcessInstance startProcessInstanceByMessageAndProcessDefinitionId(String messageName, String processDefinitionId, String businessKey);
        ProcessInstance startProcessInstanceByMessageAndProcessDefinitionId(String messageName, String processDefinitionId, Map<String, Object> processVariables);
        ProcessInstance startProcessInstanceByMessageAndProcessDefinitionId(String messageName, String processDefinitionId, String businessKey, Map<String, Object> processVariables);
        
        /**
         * 触发消息捕获事件，如果流程实例包含多个执行，则需要提供等待消息的确切执行。
         */
        void messageEventReceived(String messageName, String executionId);
        void messageEventReceived(String messageName, String executionId, Map<String, Object> processVariables);
        // 关联消息，
        MessageCorrelationBuilder createMessageCorrelation(String messageName);
        void correlateMessage(String messageName);
        void correlateMessage(String messageName, String businessKey);
        void correlateMessage(String messageName, Map<String, Object> correlationKeys);
        void correlateMessage(String messageName, String businessKey, Map<String, Object> processVariables);
        void correlateMessage(String messageName, Map<String, Object> correlationKeys, Map<String, Object> processVariables);
        void correlateMessage(String messageName, String businessKey, Map<String, Object> correlationKeys, Map<String, Object> processVariables);
        MessageCorrelationAsyncBuilder createMessageCorrelationAsync(String messageName);
    

#### 2.3 删除流程实例

`RuntimeService`提供了多种删除流程实例的方法，支持批量和异步删除。

    /**
         * 删除流程实例， 删除会在必要时向上传播。
         *
         * @param processInstanceId    流程实例ID
         * @param deleteReason         删除原因
         * @param skipCustomListeners  是否跳过自定义监听器
         * @param externallyTerminated 指示是否从外部上下文触发删除，例如REST API调用
         * @param skipIoMappings       是否跳过输入输出映射
         * @throws BadUserRequestException 当processInstanceId为null时。
         * @throws NotFoundException       当找不到具有给定processInstanceId的进程实例时。
         * @throws AuthorizationException  没有权限时
         */
        void deleteProcessInstance(String processInstanceId, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated, boolean skipIoMappings);
        void deleteProcessInstance(String processInstanceId, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated);
        void deleteProcessInstances(List<String> processInstanceIds, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated);
    
        void deleteProcessInstancesIfExists(List<String> processInstanceIds, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated,
                                            boolean skipSubprocesses);
        void deleteProcessInstance(String processInstanceId, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated, boolean skipIoMappings,
                                   boolean skipSubprocesses);
        void deleteProcessInstances(List<String> processInstanceIds, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated,
                                    boolean skipSubprocesses);
        void deleteProcessInstanceIfExists(String processInstanceId, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated, boolean skipIoMappings,
                                           boolean skipSubprocesses);
        void deleteProcessInstance(String processInstanceId, String deleteReason);
        void deleteProcessInstance(String processInstanceId, String deleteReason, boolean skipCustomListeners);
    
        // 异步批量删除
        Batch deleteProcessInstancesAsync(List<String> processInstanceIds, ProcessInstanceQuery processInstanceQuery, String deleteReason);
        Batch deleteProcessInstancesAsync(List<String> processInstanceIds, ProcessInstanceQuery processInstanceQuery, String deleteReason, boolean skipCustomListeners);
        Batch deleteProcessInstancesAsync(List<String> processInstanceIds, ProcessInstanceQuery processInstanceQuery, String deleteReason, boolean skipCustomListeners, boolean skipSubprocesses);
        Batch deleteProcessInstancesAsync(List<String> processInstanceIds,
                                          ProcessInstanceQuery processInstanceQuery,
                                          HistoricProcessInstanceQuery historicProcessInstanceQuery,
                                          String deleteReason,
                                          boolean skipCustomListeners,
                                          boolean skipSubprocesses);
        Batch deleteProcessInstancesAsync(ProcessInstanceQuery processInstanceQuery, String deleteReason);
        Batch deleteProcessInstancesAsync(List<String> processInstanceIds, String deleteReason);
    

示例代码：

        @PostMapping("/deleteProcessInstance")
        @Operation(summary = "通过ID删除流程实例")
        public R<String> startProcessInstanceById(String processInstanceId, String deleteReason, boolean skipCustomListeners, boolean externallyTerminated, boolean skipIoMappings, boolean skipSubprocesses) {
            runtimeService.deleteProcessInstance(processInstanceId, deleteReason, skipCustomListeners, externallyTerminated);
            return R.success();
        }
    

#### 2.4 变量

`RuntimeService`提供了对流程变量进行增删改查的多种方法`API`。

        /**
         * 查询指定作用域中的所有变量（包括父作用域）
         *
         * @param executionId 进程实例或执行的id不能为null。
         * @return 变量
         */
        Map<String, Object> getVariables(String executionId);
        VariableMap getVariablesTyped(String executionId);
        VariableMap getVariablesTyped(String executionId, boolean deserializeValues);
        Map<String, Object> getVariablesLocal(String executionId);
        VariableMap getVariablesLocalTyped(String executionId);
        VariableMap getVariablesLocalTyped(String executionId, boolean deserializeValues);
        Map<String, Object> getVariables(String executionId, Collection<String> variableNames);
        VariableMap getVariablesTyped(String executionId, Collection<String> variableNames, boolean deserializeValues);
        Map<String, Object> getVariablesLocal(String executionId, Collection<String> variableNames);
        Object getVariable(String executionId, String variableName);
        <T extends TypedValue> T getVariableTyped(String executionId, String variableName);
        <T extends TypedValue> T getVariableTyped(String executionId, String variableName, boolean deserializeValue);
        Object getVariableLocal(String executionId, String variableName);
        <T extends TypedValue> T getVariableLocalTyped(String executionId, String variableName);
        <T extends TypedValue> T getVariableLocalTyped(String executionId, String variableName, boolean deserializeValue);
        // 设置变量，如果重名，则是更新
        void setVariable(String executionId, String variableName, Object value);
        void setVariableLocal(String executionId, String variableName, Object value);
        void setVariables(String executionId, Map<String, ? extends Object> variables);
        void setVariablesLocal(String executionId, Map<String, ? extends Object> variables);
        Batch setVariablesAsync(List<String> processInstanceIds,
                                ProcessInstanceQuery processInstanceQuery,
                                HistoricProcessInstanceQuery historicProcessInstanceQuery,
                                Map<String, ?> variables);
        Batch setVariablesAsync(List<String> processInstanceIds, Map<String, ?> variables);
        Batch setVariablesAsync(ProcessInstanceQuery processInstanceQuery, Map<String, ?> variables);
        Batch setVariablesAsync(HistoricProcessInstanceQuery historicProcessInstanceQuery, Map<String, ?> variables);
        // 删除变量（不考虑父作用域）
        void removeVariable(String executionId, String variableName);
        void removeVariableLocal(String executionId, String variableName);
        void removeVariables(String executionId, Collection<String> variableNames);
        void removeVariablesLocal(String executionId, Collection<String> variableNames);
    

#### 2.5 流程状态

当流程实例被挂起后，流程将无法继续执行。

    // 根据ID挂起流程实例
    void suspendProcessInstanceById(String processInstanceId);
    // 挂起当前流程定义ID下所有的流程实例
    void suspendProcessInstanceByProcessDefinitionId(String processDefinitionId);
    // 挂起当前流程定义Key下所有的流程实例
    void suspendProcessInstanceByProcessDefinitionKey(String processDefinitionKey);
    // 激活具有给定id的流程实例。如果您有流程实例层次结构，从该层次结构中激活一个流程实例将不会激活该层次结构的其他流程实例
    void activateProcessInstanceById(String processInstanceId);
    void activateProcessInstanceByProcessDefinitionId(String processDefinitionId);
    void activateProcessInstanceByProcessDefinitionKey(String processDefinitionKey);
    // 统一的API去执行挂起或激活
    UpdateProcessInstanceSuspensionStateSelectBuilder updateProcessInstanceSuspensionState();
    

#### 2.6 查询

可以使用`RuntimeService`查询执行对象，可以构建查询对象，也可以通过原生`SQL`查询。

        /**
         * 创建一个新的｛@link ExecutionQuery｝实例，该实例可用于查询执行和进程实例。
         */
        ExecutionQuery createExecutionQuery();
        /**
         创建一个新的｛@link NativeExecutionQuery｝以通过SQL直接查询
         */
        NativeExecutionQuery createNativeExecutionQuery();
    

示例代码：

        @Test
        @DisplayName("查询执行对象")
        void createExecutionQuery() {
            String executionId = "1698932745109102592";
            // 普通查询
            runtimeService.createExecutionQuery()
                    .executionId(executionId) // 执行ID
                    .active();// 激活状态
            // 原生查询 
            String sql = "SELECT * FROM " + managementService.getTableName(Execution.class) +
                    " E WHERE E.ID_ ='" + executionId + "'";
            List<Execution> executionList = runtimeService.createNativeExecutionQuery().sql(sql).list();
        }
    

也提供了查询流程实例`API`，可以构建查询对象，也可以通过原生`SQL`查询。

        /**
         创建一个新的｛@link ProcessInstanceQuery｝实例，该实例可用于查询流程实例。
         */
        ProcessInstanceQuery createProcessInstanceQuery();
    
        /**
         创建一个新的｛@link NativeProcessInstanceQuery｝以直接通过SQL查询
         */
        NativeProcessInstanceQuery createNativeProcessInstanceQuery();
    

示例代码：

        @Test
        @DisplayName("查询流程实例")
        void createProcessInstanceQuery() {
            String processInstanceId = "1698932745109102592";
            // 普通查询
            runtimeService.createProcessInstanceQuery()
                    .processInstanceId(processInstanceId) // ID
                    //.processInstanceIds() // ID集合
                    .orderByProcessInstanceId().desc() // 排序
                    .active();// 激活状态
            // 原生查询
            String sql = "SELECT * FROM " + managementService.getTableName(ProcessInstance.class) +
                    " WHERE ID_ ='" + processInstanceId + "'";
            List<ProcessInstance> processInstances = runtimeService.createNativeProcessInstanceQuery().sql(sql).list();
        }
    

其他查询`API`：

        /**
         创建一个新的｛@link IncidentQuery｝实例，该实例可用于查询事件。
         */
        IncidentQuery createIncidentQuery();
    
        /**
         创建一个新的｛@link EventSubscriptionQuery｝实例，该实例可用于查询事件订阅。
         */
        EventSubscriptionQuery createEventSubscriptionQuery();
    
        /**
         * 创建一个新的｛@link VariableInstanceQuery｝实例，该实例可用于查询变量实例。
         */
        VariableInstanceQuery createVariableInstanceQuery();
    
        /**
         * 查找在活动中等待的所有执行的活动ID
         */
        List<String> getActiveActivityIds(String executionId);
    
        /**
         * 查询活动实例树
         */
        ActivityInstance getActivityInstance(String processInstanceId);
    

#### 2.7 信号启动

除了发送信息，还支持发送信号给等待该信号的所有执行，接收到信号后启动流程。

        /**
         * 通知进程引擎已接收到名为“signalName”的信号事件。将信号传递给等待该信号的所有执行，以及可以由该信号启动的所有进程定义，注意：通知和实例化同步发生。
          * @param signalName 信号名称
         */
        void signalEventReceived(String signalName);
        void signalEventReceived(String signalName, Map<String, Object> processVariables);
        void signalEventReceived(String signalName, String executionId);
        void signalEventReceived(String signalName, String executionId, Map<String, Object> processVariables);
        // 发送外号，如果流程实例包含多个执行，则需要提供等待信号的确切执行。
        void signal(String executionId);
        void signal(String executionId, String signalName, Object signalData, Map<String, Object> processVariables);
        void signal(String executionId, Map<String, Object> processVariables);
        /**
         * 通知流程引擎已使用fluent生成器接收到信号事件。
         */
        SignalEventReceivedBuilder createSignalEvent(String signalName);
    

#### 2.8 修改流程实例

支持修改流程实例，灵活地再次启动一个活动或取消一个正在运行的活动。

        /**
         * 通过流畅的构建器，根据活动取消和实例化来定义流程实例的修改。指令按照指定的顺序执行。
         *
         * @param processInstanceId 流程实例ID
         */
        ProcessInstanceModificationBuilder createProcessInstanceModification(String processInstanceId);
        // 修改多个流程实例
        ModificationBuilder createModification(String processDefinitionId);
    

示例代码：

    @Test
    @DisplayName("回退到指定节点")
    void createProcessInstanceModification() {
    String processInstanceId="d63e26e3-472a-11ee-87d5-00ff045b1bb6";
    runtimeService.createProcessInstanceModification(processInstanceId)
    .cancelAllForActivity("Activity_03g17n4") // 取消的活动实例
    .startBeforeActivity("Activity_0nc4bs3") // 当前要回到的活动实例
    .execute();
    }
    

#### 2.9 迁移流程实例

创建迁移计划以在不同的流程定义之间迁移流程实例。

        /**
         * 创建迁移计划以在不同的流程定义之间迁移流程实例。返回一个流畅的生成器，该生成器可用于指定迁移指令和生成计划。
         *
         * @param sourceProcessDefinitionId 原流程定义
         * @param targetProcessDefinitionId 目标原流程定义
         */
        MigrationPlanBuilder createMigrationPlan(String sourceProcessDefinitionId, String targetProcessDefinitionId);
    
        /**
         * 为给定的流程实例列表执行迁移计划。迁移可以同步执行，也可以异步执行。同步迁移会阻塞调用程序，直到迁移完成。只有能够迁移所有流程实例，才能成功完成迁移。
         *
         * @param migrationPlan 迁移计划
         */
        MigrationPlanExecutionBuilder newMigration(MigrationPlan migrationPlan);
    

#### 2.10 重启流程

通过历史数据重新启动流程。

        /**
         * 重新启动使用初始或最后一组变量完成或删除的流程实例。
         *
         * @param processDefinitionId 进程定义的id不能为null。
         */
        RestartProcessInstanceBuilder restartProcessInstances(String processDefinitionId);
    

#### 2.11 其他

        /**
         * 创建突发事件，例如作业、外部任务被卡主
         *
         * @param incidentType 事件类型
         * @param executionId   执行ID
         * @param configuration 配置
         */
        Incident createIncident(String incidentType, String executionId, String configuration);
        Incident createIncident(String incidentType, String executionId, String configuration, String message);
        /**
         * 解决并删除突发事件
         * @param incidentId 事件ID
         */
        void resolveIncident(String incidentId);
        // 为突发事件设置注释
        void setAnnotationForIncidentById(String incidentId, String annotation);
        // 清理注释
        void clearAnnotationForIncidentById(String incidentId);
        /**
         * 生成定义条件评估，用于触发条件事件启动流程
         */
        ConditionEvaluationBuilder createConditionEvaluation();
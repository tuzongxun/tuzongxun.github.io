---
title: activiti基础操作
date: 2017-01-30 15:19:45
tags: [activiti,java]
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
---
## 获取流程引擎
`ProcessEngines.getDefaultProcessEngine()`会在第一次调用时初始化并创建一个流程引擎，以后再调用就会返回相同的流程引擎。
使用对应的方法可以创建和关闭所有流程引擎：`ProcessEngines.init()` 和  `ProcessEngines.destroy()`。  
<!--more-->
<pre>
private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();  
</pre> 

## 部署流程定义（发布流程） 
<pre>
public void actDeployement() {  
        // 使用zip文件形式部署流程定义  
        InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("actTest1.zip");  
        ZipInputStream zipInputStream = new ZipInputStream(inputStream);  
        processEngine.getRepositoryService().createDeployment().name("activiti测试").addZipInputStream(zipInputStream).deploy();  
}  
</pre> 

## 删除流程定义
<pre>
public void deleteDeployement() {  
        List<Deployment> lists = processEngine.getRepositoryService() .createDeploymentQuery().list();  
        if (!isEmpty(lists)) {  
            for (Deployment deployment : lists) {  
                processEngine.getRepositoryService().deleteDeployment(deployment.getId());  
            }  
        }  
}  
</pre>

##  启动流程实例
<pre>
public void startProcessInstance() {  
        // 这里根据流程定义的key启动，也可以根据id，还可以在启动的时候加入流程变量,  
        // 启动流程实例后会获得一个任务task,这里是在流程图中已经写死了任务所有者是张三，因此启动的时候会创建一个任务给张三  
        String processDefinitionKey = "myProcess";  
        processEngine.getRuntimeService().startProcessInstanceByKey(processDefinitionKey);  
}  
</pre>

## 查询当前活动的流程实例
<pre>
public void findCurrentProInstance() {  
        List<ProcessInstance> lists = processEngine.getRuntimeService().createProcessInstanceQuery().list();  
        if (!isEmpty(lists)) {  
            for (ProcessInstance processInstance : lists) {  
                System.out.println(processInstance.getId());  
            }  
        }  
}  
</pre> 

## 查询个人任务及相关信息
<pre>
public void findMyTask() {  
        // String userName = "张三";  
        String userName = "李四";  
        List<Task> lists = processEngine.getTaskService().createTaskQuery() .taskAssignee(userName).list();  
        if (!isEmpty(lists)) {  
            for (Task task : lists) {  
                System.out.println(task.getId() + "," + task.getName() + "," + task.getAssignee() + "," + task.getCreateTime());  
            }  
        }  
}  
</pre>

## 完成个人任务
<pre>
public void endMyTask() {  
        String taskId = "5002";  
        processEngine.getTaskService().complete(taskId);  
}  
</pre>

## 查询历史流程实例
<pre>
public void findHisProInstance() {  
        List<HistoricProcessInstance> lists = processEngine.getHistoryService()  
                .createHistoricProcessInstanceQuery().list();  
        if (!isEmpty(lists)) {  
            for (HistoricProcessInstance hisPro : lists) {  
                System.out.println(hisPro.getId() + "," + hisPro.getStartTime()+ "," + hisPro.getEndTime());  
            }  
        }  
}  
</pre>

## 查询历史任务列表
<pre>
public void findHisTask() {  
        List<HistoricTaskInstance> lists = processEngine.getHistoryService()  
                .createHistoricTaskInstanceQuery().list();  
        if (!isEmpty(lists)) {  
            for (HistoricTaskInstance hisTask : lists) {  
                System.out.println(hisTask.getId() + "," + hisTask.getAssignee() + "," + hisTask.getName() + "," + hisTask.getStartTime() + "," + hisTask.getEndTime());  
            }  
        }  
} 
</pre> 

## 上边代码中用到的工具方法，简单非空判断
<pre>
public boolean isEmpty(Object object) {  
        if (object instanceof List) {  
            List list = (List) object;  
            if (list != null && list.size() > 0) {  
                return false;  
            } else {  
                return true;  
            }  
        } else {  
            if (object != null) {  
                return false;  
            } else {  
                return true;  
            }  
        }  
}
</pre>  
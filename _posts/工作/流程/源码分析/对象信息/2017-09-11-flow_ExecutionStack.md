---
title: 流程执行堆栈树ExecutionStack
tags: [bpmx]
---

1）执行堆栈对象

该对象主要是为了记录任务执行过程，方便回退操作的实现。如果回退任务，则会从库中查找到上一步的执行任务放到栈中。

```
public void initStack(String actInstId,List<Task> taskList,Long parentStackId){
    if(BeanUtils.isEmpty(taskList)) return;
    Map<String,ExecutionStack> nodeIdStackMap=new HashMap<String, ExecutionStack>();
    for(Task task:taskList){        
        String nodeId=task.getTaskDefinitionKey();
        if(!nodeIdStackMap.containsKey(nodeId)){
            ExecutionStack stack=new ExecutionStack();
            stack.setActInstId(new Long( actInstId));
            ...
            nodeIdStackMap.put(nodeId, stack);
        }
        else//为多实例的会签任务
        {
            ExecutionStack stack=nodeIdStackMap.get(nodeId);
            stack.setIsMultiTask(ExecutionStack.MULTI_TASK);
            stack.setAssignees(stack.getAssignees()+","+task.getAssignee());
            stack.setTaskIds(stack.getTaskIds()+ "," + task.getId());
        }
    }
    
    //保存对象到库中
    Iterator<ExecutionStack> stackIt=nodeIdStackMap.values().iterator();
    while(stackIt.hasNext()){
        ExecutionStack exeStack=stackIt.next();
        dao.add(exeStack);
    }
}
```

注：对任务进行遍历。普通的任务直接在堆栈中添加一条记录。针对会签节点会在堆栈中记录节点id，并把会签的任务id和任务执行人使用逗号分隔记录起来，再添加到堆栈中。

2）重要字段

parentId记录了该执行堆栈的上一个执行堆栈主键（可向上查找）

assignees（返回执行人IDS，如1,2,3），在流程回退时能够获取回退到流程节点时的执行人，并将该任务分配给该执行人，不需要再重新计算节点执行人的设置规则了。
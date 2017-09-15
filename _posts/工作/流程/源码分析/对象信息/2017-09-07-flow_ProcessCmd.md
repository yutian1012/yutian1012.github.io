---
title: 流程命令实体ProcessCmd
tags: [bpmx]
---

根据用户的请求信息生成命令实体

```
ProcessCmd processCmd = BpmUtil.getProcessCmd(request);
```

### ProcessCmd对象内容

1）任务对象信息：

taskId（任务ID）；
destTask（流程目标节点ActivityId，用于自由跳，及回退）；
lastDestTaskIds（下一步任务节点id，可以为多个值：'userTask1','usertask2'，封装成数组对象）；
lastDestTaskUids（最终任务处理人员，节点的执行人员，这个数组的长度和lastDestTaskIds数据的长度一致）；
taskExecutors（下一步任务执行人集合，List<TaskExecutor>）;
startNode（在发起流程时的第一个任务节点，不是startEvent节点）；
taskExecutors（List<TaskExecutor>，下一步任务执行人集合）；

注：lastDestTaskUids每个数组的值为人员，人员之间使用逗号分隔。数据：'user^用户ID^用户名,org^组织id^组织名称,role^角色ID^角色名'。

2）表单数据信息：

formData（用户提交的表达json数据）；
FormDataMap（将请求参数封装成map集合）；
businessKey（业务主键）

3）流程实例信息：

ActDefId（流程定义ID，来自Activiti的流程定义ID，act_re_procdef表）；
flowKey（流程定义Key）；
relRunId（关联流程运行ID，用于流程引用）

4）临时变量信息：

临时变量存放在transientVar集合中（Map<String,Object>）

```
//map的key值bpm_definition
cmd.addTransientVar(BpmConst.BPM_DEF, bpmDefinition);
```

5）消息提醒方式：

InformType（通知类型：1代表邮件，2代表手机短信，3代表站内消息）；

6）操作信息：

back（驳回：0正常跳转，1驳回，2驳回到发起人）；
isManage（是否管理员干预，0表示否，1表示是）

7）投票信息：

voteContent（投票意见）；
voteAgree（投票状态，0弃权，1同意，2反对，3驳回，4追回，5会签通过，6会签不通过，34=提交）；

注：voteAgree字段值信息可以从ITaskOpinion.STATUS_RECOVER常量中获取

8）执行堆栈信息：

stackId（回退时，用于把流程任务回退至stackId对应的任务Id，可为空，其值来自bpm_exe_stack的主键）；

9）流程变量信息：

```
cmd.getVariables().put(varName, valObj);
```

注：变量都是以v_开头的，变量都存放到variables集合中（Map<String, Object>）

10）执行

isOnlyCompleteTask（意思是流程任务是否只是完成。而不发生流程跳转。在执行的时候,先删除当前节点后的连线，再完成当前任务）；

11）外部调用

invokeExternal（是否外部调用，如值为true不处理自定义表单数据，不处理表单运行时情况）；

### 数据的拆分

1）获取下一步任务执行人

```
// 流程启动设置下一步执行人。
temp = request.getParameter("_executors_");

if (StringUtil.isNotEmpty(temp)) {
    List<TaskExecutor> executorList = BpmUtil.getTaskExecutors(temp);
    cmd.setTaskExecutors(executorList);
}
```

BpmUtil类的getTaskExecutiors方法

```
/**
 * 根据任务执行人字符串返回执行人列表。
 * 
 * @param executors
 *            执行人为 user^id^名称，group^id^名称
 * @return 执行人列表。
 */
public static List<TaskExecutor> getTaskExecutors(String executors) {
    String[] aryExecutor = executors.split("#");
    List<TaskExecutor> list = new ArrayList<TaskExecutor>();
    for (String tmp : aryExecutor) {
        String[] aryTmp = tmp.split("\\^");
        if (aryTmp.length == 3) {
            list.add(new TaskExecutor(aryTmp[0], aryTmp[1], aryTmp[2]));
        } else if (aryTmp.length == 1) {
            list.add(new TaskExecutor(aryTmp[0]));
        }
    }
    return list;
}
```

2）获取流程变量

```
//流程变量处理
Enumeration<String> paramEnums = request.getParameterNames();
while (paramEnums.hasMoreElements()) {
    String paramName = paramEnums.nextElement();
    if (!paramName.startsWith(VAR_PRE_NAME))//流程变量的名称是以v_开头的
        continue;
    //使用下划线作为变量名称和类型的分隔
    String[] vnames = paramName.split("[_]");
    if (vnames == null || vnames.length != 3)
        continue;
    //变量名称，数组的第一个值为v，即标识流程变量的符号
    String varName = vnames[1];
    //获取流程变量的值
    String val = (String) request.getParameter(paramName);
    if (val.isEmpty())
        continue;

    //根据流程变量的类型获取对应的值（类型转换）
    Object valObj = getValue(vnames[2], val);
    cmd.getVariables().put(varName, valObj);
}
```

BpmUtil类的getValue方法

```
/**
 * 通过变量类型及字符串值返回变量实际类型值。
 * 
 * @param type
 *            变量类型缩写有如下类型 String:S Date:D Long:L Float:F Double:DB BigDecimal:BD Integer:I Short:SH
 * @param paramValue
 * @return
 */
public static Object getValue(String type, String paramValue) {
    Object value = null;
    // 大部的查询都是该类型，所以放至在头部
    if ("S".equals(type)) {
        value = paramValue;
    } else if ("L".equals(type)) {
        value = new Long(paramValue);
    } else if ("I".equals(type)) {
        value = new Integer(paramValue);
    } else if ("DB".equals(type)) {
        value = new Double(paramValue);
    } else if ("BD".equals(type)) {
        value = new BigDecimal(paramValue);
    } else if ("F".equals(type)) {
        value = new Float(paramValue);
    } else if ("SH".equals(type)) {
        value = new Short(paramValue);
    } else if ("D".equals(type)) {
        try {
            value = DateUtils.parseDate(paramValue, new String[] { "yyyy-MM-dd", "yyyy-MM-dd HH:mm:ss" });
        } catch (Exception ex) {
        }
    } else {
        value = paramValue;
    }
    return value;
}
```
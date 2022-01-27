# ScriptRunner For Jira

## 工作流属性

```properties
jira.issue.editable = true / false表示该问题是可编辑的
jira.permission.edit.group = <任何jira组>意味着，只有属于该组的用户才能编辑问题

注：详细的属性如下。
格式：jira.permission.[subtasks.]{permission}.{type}[.suffix]
subtasks : 可选，如果想要把这个权限继承到子任务中的话就写上这个选项。
permission : JIRA对应的权限类的缩写，下面是基于JIRA4.2的可用权限类缩写列表，这个就不一一翻译了，相信如果对JIRA有一定的了解都应该知道对应的权限是什么。
admin, use, sysadmin, project, browse, create, edit, scheduleissue, assign, assignable, attach, resolve, close, comment, delete, work, worklogdeleteall, worklogdeleteown, worklogeditall, worklogeditown, link, sharefilters, groupsubscriptions, move, setsecurity, pickusers, viewversioncontrol
type : 允许/拒绝当前权限的用户，可用的值有下面几个。
group, user, assignee, reporter, lead, userCF, projectrole
suffix : 后缀，如果想要对两个用户组进行权限设置，可以通过后缀来区分。比如jira.permission.edit.group.1, jira.permission.edit.group.2
接下来在属性值中填入对应的值，比如想给某个用户组操作权限，属性值里就填用户组的名字，想给用户设置权限就在属性值中填用户的名字。
```

插件安装

## ScriptRunner

### 获取问题

```java
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.IssueManager
IssueManager issueManager = ComponentAccessor.getIssueManager()
Issue issue = issueManager.getIssueByCurrentKey("APPROVAL-6")
```

### 获取用户

```java
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.user.util.UserManager
import com.atlassian.jira.user.ApplicationUser
UserManager userManager = ComponentAccessor.getUserManager()
ApplicationUser user = userManager.getUserByName("ganzhen")
```

### 获取关注列表

```java
import com.atlassian.jira.issue.watchers.WatcherManager;
import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.user.ApplicationUser;
WatcherManager watcherManager = ComponentAccessor.getWatcherManager()
Collection<ApplicationUser> watchers = watcherManager.getWatchersUnsorted(issue)
```

### 获取自定义字段值

```java
import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.issue.CustomFieldManager;
import com.atlassian.jira.issue.fields.CustomField;
import com.atlassian.jira.user.ApplicationUser;
CustomFieldManager customFieldManager = ComponentAccessor.getCustomFieldManager();
// 10911历史经办人字段ID
CustomField historyAssineesField = customFieldManager.getCustomFieldObject(10911);
Collection<ApplicationUser> historyAssinees = (Collection<ApplicationUser>)issue.getCustomFieldValue(historyAssineesField);
```

### 更新问题

Script Console

```java
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.CustomFieldManager
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.IssueManager
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.fields.CustomField
import com.atlassian.jira.issue.fields.layout.field.FieldLayout
import com.atlassian.jira.issue.fields.layout.field.FieldLayoutItem
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder
import com.atlassian.jira.user.ApplicationUser
import com.atlassian.jira.user.util.UserManager
import com.atlassian.jira.event.type.EventDispatchOption

IssueManager issueManager = ComponentAccessor.getIssueManager()
UserManager userManager = ComponentAccessor.getUserManager()
CustomFieldManager customFieldManager = ComponentAccessor.getCustomFieldManager()
Issue issue = issueManager.getIssueByCurrentKey("APPROVAL-37")
// 设置解决结果
issue.setResolutionId("10100")
// 更新问题，不触发事件和通知
issueManager.updateIssue(issue.getReporter(),issue,EventDispatchOption.DO_NOT_DISPATCH,false)
```

### 历史经办人

Listeners

```java
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.event.issue.IssueEvent
import com.atlassian.jira.issue.CustomFieldManager
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.issue.fields.CustomField
import com.atlassian.jira.issue.fields.layout.field.FieldLayoutItem
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder
import com.atlassian.jira.user.ApplicationUser

// String eventName = ComponentAccessor.getEventTypeManager().getEventType(event.getEventTypeId()).getName()
// log.warn(eventName)
Issue issue = event.getIssue()
CustomFieldManager customFieldManager = ComponentAccessor.getCustomFieldManager()
// 10915历史经办人字段ID
CustomField historyAssigneesField = customFieldManager.getCustomFieldObject(10915)
Collection<ApplicationUser> historyAssignees = (Collection<ApplicationUser>)issue.getCustomFieldValue(historyAssigneesField)
if(historyAssignees == null){
    historyAssignees = new HashSet<ApplicationUser>()
}
if(issue.getAssignee() != null){
    historyAssignees.add(issue.getAssignee())
}
MutableIssue mutableIssue = (MutableIssue) issue
mutableIssue.setCustomFieldValue(historyAssigneesField,historyAssignees)
// 写入数据
FieldLayoutItem fieldLayoutItem = ComponentAccessor.getFieldLayoutManager().getFieldLayout(issue).getFieldLayoutItem(historyAssigneesField)
Map<String, ModifiedValue> modifiedFields = mutableIssue.getModifiedFields()
ModifiedValue modifiedValue = modifiedFields.get(historyAssigneesField.getId())
DefaultIssueChangeHolder issueChangeHolder = new DefaultIssueChangeHolder()
historyAssigneesField.updateValue(fieldLayoutItem,mutableIssue,modifiedValue,issueChangeHolder)
// log.warn("success")
```

### 清空历史经办人

Post Function

```java
import com.atlassian.jira.security.groups.GroupManager
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.CustomFieldManager
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.fields.CustomField
import com.atlassian.jira.issue.fields.layout.field.FieldLayoutItem
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder
import com.atlassian.jira.user.ApplicationUser

// 设置历史经办人
GroupManager groupManager = ComponentAccessor.getGroupManager()
Collection<ApplicationUser> users = new HashSet<ApplicationUser>()

CustomFieldManager customFieldManager = ComponentAccessor.getCustomFieldManager()
// 10915 历史经办人字段
CustomField historyAssigneesField = customFieldManager.getCustomFieldObject(10915)
issue.setCustomFieldValue(historyAssigneesField,users)
```

### 设置抄送人

Post Function

```java
import com.atlassian.jira.security.groups.GroupManager
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.CustomFieldManager
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.fields.CustomField
import com.atlassian.jira.issue.fields.layout.field.FieldLayoutItem
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder
import com.atlassian.jira.user.ApplicationUser

// 设置抄送人
GroupManager groupManager = ComponentAccessor.getGroupManager()
Collection<ApplicationUser> users = groupManager.getUsersInGroup("财务中心",false)

CustomFieldManager customFieldManager = ComponentAccessor.getCustomFieldManager()
// 10916 抄送人字段
CustomField CCListField = customFieldManager.getCustomFieldObject(10916)
issue.setCustomFieldValue(CCListField,users)
```

### 设置审批人为部门主管

Post Function 注：需要用户组中有 XX 和 XX 主管组

```java
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.IssueManager
import com.atlassian.jira.user.ApplicationUser
import com.atlassian.jira.user.util.UserManager
import com.atlassian.jira.bc.group.search.GroupPickerSearchService
import com.atlassian.jira.bc.group.search.GroupPickerSearchServiceImpl
import com.atlassian.crowd.embedded.api.Group
import com.atlassian.jira.security.groups.GroupManager

UserManager userManager = ComponentAccessor.getUserManager()
ApplicationUser reporter = null
reporter = issue.getAssignee()
GroupManager groupManager = ComponentAccessor.getGroupManager()
GroupPickerSearchService groupPickerSearchService = new GroupPickerSearchServiceImpl(userManager)
ApplicationUser assignee = null
List<Group> groups = groupPickerSearchService.findGroups("主管")
for(int i=0;i<groups.size();i++){
    Group group = groups[i]
    String groupName = group.getName()
    Collection<ApplicationUser> usersInGroup = groupManager.getUsersInGroup(groupName,false)
    // 主管组没有激活的人，跳过
    if(usersInGroup.size()<1){
        continue
    }
    String subGroupName = groupName.substring(0,groupName.length()-2)
    Group subGroup = groupPickerSearchService.getGroupByName(subGroupName)
    // 找不到对应的非主管组，跳过
    if(subGroup == null){
        continue
    }
    // 报告人不在非主管组，跳过
    if(! groupManager.isUserInGroup(reporter,subGroup)){
        continue
    }
    assignee = usersInGroup[0]
    break
}
issue.setAssignee(assignee)
```

### 批量子任务

```java
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.IssueFactory
import com.atlassian.jira.issue.IssueManager
import com.atlassian.jira.user.ApplicationUser
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.security.groups.GroupManager
import com.atlassian.jira.config.SubTaskManager

IssueManager issueManager = ComponentAccessor.getIssueManager()
GroupManager groupManager = ComponentAccessor.getGroupManager()
IssueFactory issueFactory = ComponentAccessor.getIssueFactory()
SubTaskManager subTaskManager = ComponentAccessor.getSubTaskManager()

Issue issue = issueManager.getIssueByCurrentKey("OPTS-109")
Issue parentIssue = issue

if (parentIssue.getIssueType().isSubTask())
 return

Collection<ApplicationUser> users = groupManager.getUsersInGroup("一键四连",false)
for(ApplicationUser user : users){
    MutableIssue newSubTask = issueFactory.getIssue()
    newSubTask.setProjectObject(parentIssue.getProjectObject())
    newSubTask.setIssueTypeId("10101")
    newSubTask.setParentObject(parentIssue)
    newSubTask.setSummary(parentIssue.getSummary())
    newSubTask.setDescription(parentIssue.getDescription())
    newSubTask.setPriority(parentIssue.getPriority())
    newSubTask.setAssignee(user)
    newSubTask.setReporter(parentIssue.getAssignee())

    issueManager.createIssueObject(parentIssue.getAssignee(), newSubTask)
    subTaskManager.createSubTaskIssueLink(parentIssue,newSubTask,parentIssue.getAssignee())
}
```

### JQL 搜索问题

```groovy
import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.jql.parser.JqlQueryParser;
import com.atlassian.jira.issue.search.SearchProvider;
import com.atlassian.jira.web.bean.PagerFilter;
import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.issue.search.SearchQuery;

def findIssues(String jqlQuery,String username) {
  def issueManager = ComponentAccessor.issueManager;
  def userManager = ComponentAccessor.userManager;
  def user = userManager.getUserByName(username);
  def jqlQueryParser = ComponentAccessor.getComponent(JqlQueryParser);
  def searchProvider = ComponentAccessor.getComponent(SearchProvider);
  def query = jqlQueryParser.parseQuery(jqlQuery);
  def searchQuery = SearchQuery.create(query, user);

  def results = searchProvider.search(searchQuery, PagerFilter.getUnlimitedFilter());
  log.warn "issue cnt: ${results.getTotal()}";
  results.getResults().collect { res ->
    def doc = res.getDocument();
    def key = doc.get("key");
    def issue = ComponentAccessor.getIssueManager().getIssueObject(key);
    return issue;
  }
}

def jqlQuery = "resolution = Unresolved ORDER BY priority DESC, updated DESC";
def issues = findIssues(jqlQuery, 'zhxlp');


```

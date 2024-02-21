# Dynamics 365 Plugin SharedVariables

In context of Plugin development, SharedVariables is really useful when we need pass data from pre-event to post event, regarding the same message.

We could benefit the plugin SharedVariables when need to pass a retrieving/checking result, for ex, suppose both pre and post events needs the security roles of the record owner, we can retrieve all the security roles of owner as a List<string> and pass it to post-event, in post-event, the code can reuse this security roles list directly instead of retriving it again.
Another senario is that the required information we can only get on pre-event, not post-event. For ex, we would like to log certain record information deleted by user, as the record is deleted when arriving at post-delete event, we need to prepare the record information in pre-delete event and than pass it to post-delete event.

let's take delete lead as example. 
For pre-delete lead operation, we retrieve the lead information and wrap it in SharedVariables

```cs
var leadEntity = service.Retrieve("lead", context.PrimaryEntityId, new ColumnSet("subject"));
var leadName = leadEntity.GetAttributeValue<string>("subject");
context.SharedVariables["DeleteLeadInfo"] = $"The lead record with Id {leadEntity.Id} and name [{leadName}] was deleted";
```

For post-delete lead event, (registed as async mode because the plugin actions are not required to be executed immediatly), plugin gets the SharedVariables from execution context and proceed them.

```cs
 if (context.SharedVariables.Contains("DeleteLeadInfo"))
{
    var info = context.SharedVariables["DeleteLeadInfo"]?.ToString();
    tracingService.Trace(info);
}
```
Above example plugin traces the deleted lead informaiton, we could also perform diffrent actions as per the requiremnt, like sending an email or writing audit log as a specific entity.

Hope SharedVariables helps you to implement plugin solution.
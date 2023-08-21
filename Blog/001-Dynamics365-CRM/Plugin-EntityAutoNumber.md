# Dynamics 365 Plugin Entity Auto Number

Auto Number is a very basic and common requirement for CRM implementation. Even though there is out of box auto number field but it can not satisfy some requirements, such as restart the seq number every day.
The challenge is that how to ensure the uniqueness of seq number. Here, I follow the principle of this article [implement-robust-auto-numbering](https://us.hso.com/blog/how-to-implement-robust-auto-numbering-using-transactions-in-microsoft-dynamics-crm/) to implement a personal version of auto number.
I designed 2 entities:

* Auto Number Config : Contains the related business entity, the format of auto number and etc.
* Auto Number Counter :  Contains the current seq number, a unique key value indicating the related entity name and restart frequency, this record associates to an auto number config record.

![Auto Number Config Form](/Blog/001-Dynamics365-CRM/Asset-Images/Plugin-EntityAutoNumberConfigForm.png)

![Auto Number Counter Form](/Blog/001-Dynamics365-CRM/Asset-Images/Plugin-EntityAutoNumberCounterForm.png)

I take the entity Order for example. Create an auto number plugin, registered on pre-create event.
For this example, this auto number contains a "Dynamic" prefix as per the code of factory associated to the order, which is configured by Prefix Type and Prefix Value. AutoNumber text contain the information from a lookup value is also common requirement, such as by user area, by business line etc, you could find appropriate design to extend the configuration.
I have designed the restart frequency with 4 values, by year, by month, by day and empty(never restart), if restart the counter by day, we set the key name as below

```cs
 var counterKey = $"{targetEntity.LogicalName.ToLower()}{restartKey}";
```

when getting the next seq number,  we search the counter by key name, if not found, create a new, else, we update the dumy field (here, I update it with a random Guid value) and then retrieve again the current seq number.

```cs
private void LockAutoNumberCounter(IOrganizationService service, Guid autoNumberCounterId)
{
    var entity = new Entity("new_autonumbercounter", autoNumberCounterId);
    entity["new_lock"] = Guid.NewGuid().ToString();
    service.Update(entity);
}
```

```cs

if (counterEntity == null)
{
    CreateAutoNumberCounter(service, counterKey, autoNumberEntity);
}
else
{
    //update the counter record so that it will be locked
    LockAutoNumberCounter(service, counterEntity.Id);
    //retrieve again the current seq number
    counterEntity = service.Retrieve("new_autonumbercounter", counterEntity.Id, new ColumnSet("new_seqnumber"));
    newSeqNumber = counterEntity.GetAttributeValue<int>("new_seqnumber") + 1;
    UpdateAutoNumberCounterSeqNumber(service, counterEntity.Id, newSeqNumber);
}
```

finally, we assigned the new seq number to record and increment the counter record.

Registered the plugin, prepare a console application with multi thread to simulate the concurrent creation of order, no duplication of order number.

![Order View](/Blog/001-Dynamics365-CRM/Asset-Images/Plugin-ActiveOrderViewWithOrderNumber.png)

the next day, create new order and checked the counter records, there are 2 counter records.

![Auto Number Counter View](/Blog/001-Dynamics365-CRM/Asset-Images/Plugin-AutoNumberCounterView.png)

Please note that in case of restart the seq number, please pay attention of the UTC time and local time, as plugin executed on UTC time, and restart by day, requires the conversion to the local time.

For complete code, please go to [Github EntityAutoNubmer](https://github.com/gaosongfour/Dynamics365-Coding/blob/master/Plugins/Crm.Plugin.AutoNumber/Crm.Plugin.AutoNumber/EntityAutoNubmer.cs)

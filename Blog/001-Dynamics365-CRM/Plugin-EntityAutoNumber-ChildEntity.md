# Dynamics 365 Plugin Entity Auto Number For Child Entity

The previous article describes an auto number solution for entity. Now, we need to auto-number for a child entity, like order item (order details), and we follow the SAP pattern, 10,20,30..., here is my solution.
As usual, I create a dedicated auto number config record for order item,  the scope of seq number for order items is order-wide, so I add 2 new attributes on auto number config entity indicating that the scope is set to Parent Entity and the relevant parent lookup id which will be used as a key for the counter

![Auto Number Config Form Add Parent Lookup](/Blog/001-Dynamics365-CRM/Asset-Images/Plugin-EntityAutoNumberConfigForm-ParentLookup.png)

Secondly, add 2 new attributes on auto number config, initial number and increment by, so that for our case, we set it 10 and 10.  For existing config of order, the value are 1 and 1 respectively.

![Auto Number Config Form Start Seq Number](/Blog/001-Dynamics365-CRM/Asset-Images/Plugin-EntityAutoNumberConfigForm-StartSeqNumber.png)

The plugin code was updated to include above enhancements

```cs
 var newSeqNumber = autoNumberEntity.GetAttributeValue<int>("new_startseqnumber");
 newSeqNumber = counterEntity.GetAttributeValue<int>("new_seqnumber") + autoNumberEntity.GetAttributeValue<int>("new_incrementby");

//Parent lookup, used for child entity numbering
case 100:
    var parentLookupAttributeName = autoNumberEntity.GetAttributeValue<string>("new_parentlookupattributename");
    if (string.IsNullOrEmpty(parentLookupAttributeName))
        throw new InvalidPluginExecutionException("Parent lookup attribtue name not definied");
    if (!targetEntity.Contains(parentLookupAttributeName))
        throw new InvalidPluginExecutionException($"Parent lookup value {parentLookupAttributeName} for record {targetEntity.LogicalName}{targetEntity.Id} is null");
    restartKey = targetEntity.GetAttributeValue<EntityReference>(parentLookupAttributeName).Id.ToString();
    break;
```

then register new step for pre-create event of Order Item.
Run an application to bulk create the order items, the seq number were generated successfully

![Auto Number Order Items View](/Blog/001-Dynamics365-CRM/Asset-Images/Plugin-OrderItemView.png)
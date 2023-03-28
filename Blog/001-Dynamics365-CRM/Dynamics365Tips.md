# Dynamics 365 Sales/CRM - Some Tips

## Lookup Attribute

It is recommended that using "id" as the end of the lookup attribute, and include the target entity name, like `{prefix}_{targetEntityName}id`, for example, there is a custimized entity name new_city, so the lookup attribute name could be `new_cityid`. Sometimes, the primary entity could have multiple lookup attributes pointing to the same target entity, we could includ business requirement when design the attribute name, for example, `new_preferred_cityid`,`new_shipto_cityid`.

## OptionSet(Picklist)

When creating a optionset attribute(or global optionset value), sometimes it is not a best practice to make the value(index) sequentially like 1,2,3,4.... because there is no reserved index for any changes in the future

```text
10 - Draft
20 - Pending for Approval
100 - Approved
999 - Rejected
```

If user would like to rename the status "Pending for Approval" to "Pending for approval (CRM)" and "Pending for approval (finance system)", we can add the new option indexed 21.

We could also define the optionset value for some sepecific senarios
for the year/month, we could define value sames as the year/month, this may facitate the converting betweenn label and value

```text
2022 - 2022
2023 - 2023
2024 - 2024
```

```text
1 - Jan
2 - Feb
12 - Dec
```

## Use a new entity for updating operation and set only the required attributes to be updated

For update operation, do not use an entity comes from retrieved operation or plugin image, the retrieved entity may include extra attributes which are not to be updated, and will lead to trouble, for example, if there is plugin/workflow monitoring the attribute "name", then this plugin/workflow will be triggered unexpectedlly . So below code should be avoided.

```cs
            var accountEntity = service.Retrieve("account", accountId, new ColumnSet("name", "accountnumber"));
            //Do not update an entity which is retrieved by service
            accountEntity["description"] = "Do not";
            service.Update(accountEntity);
```

Use a new entity for update opreation

```cs
     //New entity to update
            var entity = new Entity(accountEntity.LogicalName, accountEntity.Id);
            entity["description"] = "new entity";
            service.Update(entity);
```

## Set required attributes for retrieve operation and for Plugin Image

Sometimes, it is required only a few attributes for retrieve/retrieve multiple operations(IOrganizationService, WebAPI, fetchXml), it's better to list all required attributes as below

```cs
var cols= new ColumnSet("name", "accountnumber","XXX");
```

Instead of  `var colsAll = new ColumnSet(true);`

if only need to get the entity count, can simply use `new ColumnSet(false)`

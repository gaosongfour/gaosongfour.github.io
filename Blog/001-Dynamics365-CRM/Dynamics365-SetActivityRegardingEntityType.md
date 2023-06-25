# Dynamics 365 Sales/CRM Set Activity Regarding Entity Type

Recently, my organization start to use out of box sales activities, such as phonecall and appoinment, but for the field `Regarding`, there are lots of entities availble for the selecton but not required by business, this will make users confused and take more time to complete an activity, so it is requested to narrow down the entity types list.

Firstly, I follow the offcial document to set entity types, as below

```js
const entityTypes = ["account", "contact", "opportunity", "custom_entity", "custom_entity2"];
formContext.getControl("regardingobjectid").setEntityTypes(entityTypes);
```

It works well on Dev but an error like "invaid entity type {custom_entity} xxx" on UAT, so I am trying an inner join the below code

```js
const expectedEntityTypes = ["account", "contact", "opportunity", "custom_entity", "custom_entity2"];
const defaultEntityTypes = formContext.getControl("regardingobjectid").getEntityTypes();
const entityTypes = expectedEntityTypes.filter(entityName => defaultEntityTypes.includes(entityName));
formContext.getControl("regardingobjectid").setEntityTypes(entityTypes);
```

It resolved the error message issue but the custom_entity is still not availble in the list. Checked the default entity types, it's ture that there is a gap between these 2 environments, finally I found something on this link [power-platform-settings-features](https://learn.microsoft.com/en-us/power-platform/admin/settings-features),
there is a setting named `Activities`, the description is:
>Default: Off. Select On to limit the number of activities showing up in the New Activity dropdown list to activities that are relevant to the model-driven app.

I follow this idea and open the UCI App that I am using, checked the relavent entities, and Yes, there is no custom_entity in the list. Aftering adding this entity and publish the App, I can select this entity in the dropdown list.

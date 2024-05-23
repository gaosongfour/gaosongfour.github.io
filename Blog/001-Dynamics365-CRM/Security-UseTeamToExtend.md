# Use Team to extend role-based security

Recently, I find a good example that uses CRM Team to extend user privileges in a flexible way.

Let's take a look at the Business Unit Hierarchy as below, the organization is firstly divided by their buinsess line then by region, at the bottom, we place sales users who has only user-level access right on entity Account, as per the security role assignd.

![BU Hierarchy](/Blog/001-Dynamics365-CRM/Asset-Images/SecurityRole-UseTeamToExtend.svg)

The requirement needs that all the accounts in the Business Line A shall be viewed by all the sales users, but they can not read the accounts in other Business Line. The solution is that we can create a specific Team belonging to the BU Business Line A, create a new security role that defines the account read privileges as BU&Sub-BU level. Last step is to add all the Sales Users to this Team.

Voilà.

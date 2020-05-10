## API v2 Schemas

This file lists the schemas for the data returned from Deepthought

### Plan
When the plan is hydrated from WSAPI, the groups, backlogItems and WorkItems are all hydrated. Due to performance implications, WSAPI data is only hydrated when explicitly `GET`ing a plan by `id` (see [api-endpoints](doc/api-endpoints.md)).

#### Unhydrated Plan
<pre>
{
_id: ObjectId // Unique identifier for this record
formattedId: string
planName: string (updatable)
startRelease: UUID (updatable)
endRelease: UUID (updatable)
projectScopeId: UUID (updatable)
itemPlanningLevel: UUID (refers to typedef features or initatives) (updatable)
isPublished: boolean (updatable)
createdBy: {
userId: UUID
userDisplayName: string
userEmail: string
timeStamp: DateTime
}
lastUpdate: {
userId: UUID
userDisplayName: string
userEmail: string
timeStamp: DateTime
}
// Only populated if the plan has been published at least once
publishedBy: {
userId: UUID
userDisplayName: string
userEmail: string
timeStamp: DateTime
}
cutlinePosition: Integer (updatable), index into backlogItems, indexes the
item which the cutline should be placed above. To place the
cutline at the bottom the index should be equal to the
number of backlog items. May be set to null. If null the
client should place the cutline in the view based on
capacity.
assignments: [
<strong><a href="#assignment">Assignments</a></strong>
]
groups: [
<strong><a href="#unhydrated-group">Unhydrated Groups</a></strong>
]
backlogItems: [
<strong><a href="#unhydrated-backlogitem">Unhydrated BacklogItems</a></strong>
]
releases: {
startRelease: WSAPI hydrated object for start release
endRelease: WSAPI hydrated object for end release
}
}
</pre>

#### Hydrated Plan
<pre>
{
... everything from the Unhydrated Plan above with these overrides...

groups: [
<strong><a href="#hydrated-group">Hydrated Groups</a></strong>
]
backlogItems: [
<strong><a href="#hydrated-backlogitem">Hydrated BacklogItems</a></strong>
]
workItems: [ // DEPRECATED
<strong><a href="#workitem---depreciated">Work Items</a></strong>
]
planningItemType: WSAPI hydrated object for itemPlanningLevel
}
</pre>

### Group
> Groups represent the teams and/or delivery groups associated with the plan

#### Unhydrated Group
```
{
groupId: ObjectIDÊ // Unique identifier for this record
almItemId: UUID
estimatedCapacity: int (updatable)
name: string (updatable)
type: string - one of #{"team", "delivery-group"}
}
```
?
#### Hydrated Group
<pre>
{
... everything from Unhydrated Group above ...
?
_ref: from WSAPI
_refObjectUUID: from WSAPI
objectID: from WSAPI
expertiseCapacity: [ // If available
<strong><a href="#expertise-capacity">Expertise Capacity</a></strong>
]
}
</pre>

### Assignment
Assignments associate [groups](#group) and [backlog items](#backlog-item)

```
{
assignmentId: ObjectID // Unique identifier for this record
almItemId: UUID
groupId: ObjectID
groupEstimate: double (updatable)
}
```
?
### BacklogItem
 BacklogItems represent the selected Portfolio Items in the plan

#### Unhydrated BacklogItem
```
{
itemId: ObjectID // Unique identifier for this record
almItemId: UUID (updatable)
rank: string (insertable but not updatable)
index: long (not updatable, provided for convenience)
}
```
?
#### Hydrated BacklogItem
<pre>
{
... everything from Unhydrated BacklogItem above ...

_ref: from WSAPI
_refObjectUUID: from WSAPI
displayColor: from WSAPI
dragAndDropRank: from WSAPI
formattedID: from WSAPI
leafStoryCount: from WSAPI
leafStoryPlanEstimateTotal: from WSAPI
name: from WSAPI
objectID: from WSAPI
parent: from WSAPI
predecessors: from WSAPI
preliminaryEstimate: from WSAPI
project: from WSAPI
refinedEstimate: from WSAPI
successors: from WSAPI
value: from WSAPI
expertiseDemand: [ // If available
<strong><a href="#expertise-demand">Expertise Demand</a></strong>
]
}
</pre>

### ExpertiseCapacity
```
{
amount: int
expertise: from WSAPI
_ref: from WSAPI
_refObjectUUID: from WSAPI
}
```

### ExpertiseDemand
```
{
ÊÊamount: int
ÊÊexpertise: from WSAPI
ÊÊ_ref: from WSAPI
ÊÊ_refObjectUUID: from WSAPI
}
```
?
### ~~WorkItem~~ - Deprecated
> WorkItems were used when the backlog items came straight from WSAPI, and weren't stored in Deepthought
?
<pre>
{
_ref: from WSAPI
_refObjectUUID: from WSAPI
displayColor: from WSAPI
dragAndDropRank: from WSAPI
formattedID: from WSAPI
leafStoryCount: from WSAPI
leafStoryPlanEstimateTotal: from WSAPI
name: from WSAPI
objectID: from WSAPI
parent: from WSAPI
predecessors: from WSAPI
preliminaryEstimate: from WSAPI
project: from WSAPI
refinedEstimate: from WSAPI
successors: from WSAPI
value: from WSAPI
expertiseDemand: [
<strong><a href="#expertise-demand">Expertise Demand</a></strong>
]
}
</pre>


# Endpoints

## Deepthought API v2 Endpoints

All endpoints that return data put it on the `result` key in a JSON response.

### Plan methods
Function | Method | URI | Parameters | Notes | Response Result |
|--------|--------|-----|------------|-------|-----------------|
Query for plans | GET | /v2/plan? | Valid fields: `startRelease`, `endRelease`, `projectScopeId`, `planName`Ê | See field types [here](api-schema-2.md) | Array of [unhydrated plans](api-schema-2.md#plan)
Get a single plan | GET | /v2/plan/{id} | id of the plan | | [Hydrated plan](api-schema-2.md#plan)
Create a new plan | POST | /v2/plan | | JSON body only needs fields to be specified | [Unhydrated plan](api-schema-2.md#plan)
Update a plan | PUT | /v2/plan{id} | id of the plan | Body should only contain the fields needing to be updated | [Unhydrated plan](api-schema-2.md#plan)
Delete a plan | DELETE | /v2/plan/{id} | id of the plan | Only marks the plan deleted | `200` status code marks success
Copy an existing plan | POST | /v2/plan/{id}/copy | id of the plan to copy | Copied plan name is prefixed with `(Copy of) ` and will be unpublished | [Unhydrated plan](api-schema-2.md#plan)

### Group methods
Function | Method | URI | Parameters | Notes | Response Result |
|--------|--------|-----|------------|-------|-----------------|
Create a group | POST | /v2/plan/{id}/group | id of the plan | | [Unhydrated group](api-schema-2.md#unhydrated-group)
Update a group | PUT | /v2/plan/{id}/group/{groupId} | `id` - id of the plan<br/><br/>`groupId` - id of the group | Body should contain JSON object with the following keys: `type`, `almItemId`, `name` | [Unhydrated group](api-schema-2.md#unhydrated-group)
Delete a group | DELETE | /v2/plan/{id}/group/{groupId} | `id` - id of the plan<br/><br/>`groupId` - id of the group | | `200` status code marks success

### Assignment methods
Function | Method | URI | Parameters | Notes | Response Result |
|--------|--------|-----|------------|-------|-----------------|
Create an assignment | POST | /v2/plan/{id}/assignment | `id` - id of the plan | | [Assignment](api-schema-2.md#assignment)
Update an assignment | PUT | /v2/plan/{id}/assignment/{assignmentId} | `id` - id of the plan <br/><br/>`assignemntId` - id of the assignment | Body should only contain the fields needing to be updated | [Assignment](api-schema-2.md#assignment)
Delete an assignment | DELETE | /v2/plan/{id}/assignment/{assignmentId} | `id` - id of the plan <br/><br/>`assignemntId` - id of the assignment | | `200` status code marks success

### Backlog methods
Function | Method | URI | Parameters | Notes | Response Result |
|--------|--------|-----|------------|-------|-----------------|
Create backlog item(s) | POST | /v2/plan/{id}/backlog | `id` - id of the plan | Body is either a single back log item or an array of backlog items. Only unique backlog items will be created. The passed rank is used to sort backlog items until a call to `rank-above` is made then new items are ranked lowest. | Array of [backlog `itemId`s](api-schema-2.md#backlogitem) added
Delete a backlog item | DELETE | /v2/plan/{id}/backlog/{backlogId} | `id` - id of the plan <br/><br/>`backlogId` - id of the backlog item | | `200` status code marks success
Bulk delete backlog items | DELETE | /v2/plan/{id}/backlog | `id` - id of the plan | Body is a JSON array of `itemId`s that identify backlog items to remove from the plan. This method is idempotent. | Array of [backlog `itemId`s](api-schema-2.md#backlogitem) removed
Rank a backlog item above another | POST | /v2/plan/{id}/backlog/{backlogId}/rank-above/{lowerId} | `id` - id of the plan <br/><br/>`backlogId` - id of the backlog item<br/><br/>`lowerId` - id of the backlog item to rank this item above. Optionally set the cutline position by sending a JSON encoded body that includes the cutline index as in the following example, `{"cutlinePosition":2}`. | If lowerId is not given then the backlog item will be ranked lowest. | `200` status code marks success
Assign backlog item to 0 or more groups. | PUT | /v2/plan/{id}/backlog/{backlogId}/assignments | `id` - id of the plan <br/><br/>`backlogId` - id of the backlog item | Only `groupId` and `groupEstimate` need to be sent for an assignment. Overwrites the current assignments for this backlog item | Array of new assignments

### Miscellaneous methods
Function | Method | URI | Parameters | Notes | Response Result |
|--------|--------|-----|------------|-------|-----------------|
Healthcheck | GET | /healthcheck | | No auth required |
Plan Stats | GET | /stats | | Super admin access only |
Metrics | GET | /metrics | | Full dump of JMX metrics |
Prometheus | GET | /prometheus | | produces prometheus metrics |

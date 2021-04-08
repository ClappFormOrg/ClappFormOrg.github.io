# Query
The query editor is meant to create a pipeline that transforms data at the latest possible stage before presentation. This means it is intended for selecting only the data necessary for creating a widget. Some types have the query disabled as the widget doesn't need any input data. Only the developer needs rights to certain data in order to put it in the pipeline. The data will not be user specific and therefore not adhere to any set permissions.
# Pipeline
The top layer of the query editor consists of the stages together forming the pipeline. These stages can be used in any quantity and order, but caution is advised for the sake of time-to-live of the widget. At the start of the pipeline the data will be passed down of the collection selected in the form. The data for the last stage will be passed down to the widget and can then be configured in the widget configuration. The last stage will always be limited to 100 items maximum to guarantee performance. The stages can have multiple different types:
*	left join
*	show
*	limit
*	order
*	group
*	filters
*	kpis
# Left join
This type is intended for joining data from a collection within the same app. This would be used if the data necessary for a table is split up in 2 or more collections (i.e. employee and manager). Below are the possible options to pass to this stage.

Name | Value
--- | ---
collection | Text
how |	Text (inner or outer, defaults to inner)
left_key |	Text (key in the previous stage)
right_key |	Text (key of the selected collection)

#### Example:

```json
{
  "type": "left join",
  "how": "outer",
  "collection": "1017",
  "left_key": "adress",
  "right_key": "adress"
}
```
# Show
This type is used for selecting keys to pass to the next stage.

Name | Value
--- | ---
columns	| Array (List containing names of the keys)

#### Example:

```json
{
  "type": "show",
  "columns": ["new_acceptance", "adress", "building_year", "house_price", "postcode"]
}
```
 
# Limit
This type will limit the amount of items passed down to the next stage.

Name | Value
---|---
amount |	Number

# Order
The order stage allows you to order all the items passed down on multiple fields. Therefore this stage contains a 'steps' field which is a list that allows for multiple keys to be declared. Every step in this list must contain the following keys:

Name | Value
---|---
field	| Text
how	| Text (ASC or DESC, defaults to DESC)

#### Example:

```json
{
  "type": "order",
  "steps": [{
      "field": "postcode",
      "how": "DESC"
  }]
}
```
# Group
In order to create calculated values over multiple items the group stage exists. In this stage the group keys must be declared together with the calculated keys it must retain. All keys not set here will not be passed down. 

Name | Value
---|---
by | Text (Optional)
fields | Object

The field needs to get a name, type of calutation and the column on which to perform the calculation.
The possible options for the type of fields include:

 - sum 
 - avg (Average/ median)
 - min (Minimum)
 - max (Maximum)
 - sdp (Standard Deviation Population)
 - sds (Standard Deviation Sample)
 - first (First value)
 - last (Last value)
 - list 
 - ulist (Unique List)
 - count (Total values)
 - ucount (Total of unique values)
 
#### Example:

```json
{
  "type": "group",
  "by": ["postcode"],
  "fields": { 
      "ucount": {
          "type": "ucount",
          "column": "house_price"
      },
      "address": {
          "type": "first",
          "column": "adress"
      }
  }
}
```

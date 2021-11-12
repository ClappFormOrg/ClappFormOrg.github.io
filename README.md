# Query
The query editor is meant to create a pipeline that transforms data at the latest possible stage before presentation. This means it is intended for selecting only the data necessary for creating a widget. Some types have the query disabled as the widget doesn't need any input data. Only the developer needs rights to certain data in order to put it in the pipeline. The data will not be user specific and therefore not adhere to any set permissions.
# Pipeline
The top layer of the query editor consists of the stages together forming the pipeline. These stages can be used in any quantity and order, but caution is advised for the sake of time-to-live of the widget. At the start of the pipeline the data will be passed down of the collection selected in the form. The data for the last stage will be passed down to the widget and can then be configured in the widget configuration. The last stage will always be limited to 100 items maximum to guarantee performance. The stages can have multiple different types:
*	`left join`
*	`show`
*	`rename`
*	`limit`
*	`order`
*	`group`
*	`filters`
*	`calculation`
*	`kpis`

#### A full example could be
```json
[
  {
    "filters": [
      {
        "regio": [
          {
            "type": "eq",
            "value": "Nederland"
          }
        ]
      }
    ],
    "type": "filters"
  },
  {
    "steps": [
      {
        "field": "periode",
        "how": "DESC"
      }
    ],
    "type": "order"
  },
  {
    "amount": 1,
    "type": "limit"
  }
]
```

# Values
If a value needs to be added to the query one may use the following types: string (text), number or value from a key. Text needs to be surrounded by double quotes and to select the value from a key the name needs to be proceeded by a $ sign. For example `"value": "$price"`. 

# Global Values
At this time of writing there are several global variables that can be used in the query builder. `"$USER"` will be replaced with the id of the user in the database. This way you can make dashboards user-specific. `"$DEEP_DIVE"` provide access to the values of a chart when you click on this specific chart. the key will be replaced with the actual value. For exmaple if the chart contains information regarding an address and you want to use this key value use: `"$DEEP_DIVE.address"`. The last global varaible is the `"$ID"`, using this the id in the url will be extracted and replaced in the database with this value. 



# Stage types
## Left join
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
## Show
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
## Rename
This type is used for renaming the keys to pass to the next stage.

Name | Value
--- | ---
fields	| Object(Where the key is the new name and the value is the old name)

#### Example:

```json
{
  "type": "rename",
  "fields": {
    "new": "old"
  }
}
```
 
## Limit
This type will limit the amount of items passed down to the next stage.

Name | Value
---|---
amount |	Number

## Order
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
## Group
In order to create calculated values over multiple items the group stage exists. In this stage the group keys must be declared together with the calculated keys it must retain. All keys not set here will not be passed down. 

Name | Value
---|---
by | List (Keys that together need to form a new unique item, Optional)
fields | Object

The field needs to get a name, type of calutation and the column on which to perform the calculation.
The possible options for the type of fields include:
 * `sum`
 * `avg` (Average/ median)
 * `min` (Minimum)
 * `max` (Maximum)
 * `sdp` (Standard Deviation Population)
 * `sds` (Standard Deviation Sample)
 * `first` (First value)
 * `last` (Last value)
 * `list`
 * `ulist` (Unique List)
 * `count` (Total values)
 * `ucount` (Total of unique values)
 
#### Example:

```json
{
  "type": "group",
  "by": ["postcode"],
  "fields": { 
      "total_unique": {
          "type": "ucount",
          "column": "house_price"
      },
      "address": {
          "type": "first",
          "column": "address"
      }
  }
}
```

## Filters
The filters stage's intended purpose is to query the data and return all matching records. One can i.e. select only country specific data. There is support for AND statements and OR statements on multiple fields. Every expression needs to contain a `type` and `value`.

Different types are:
* `eq` (Equal to)
* `ne` (Not equal to)
* `lt` (Less than)
* `lte` (Less than or equal to)
* `gt` (Greater than)
* `gte` (Greater than or equal to)

#### Examples:
##### Where region equals Nederland and price is greater than 1000
```json
{
  "filters": [
    {
      "regio": [
        {
          "type": "eq",
          "value": "Nederland"
        }
      ],
      "price": [
        { 
          "type": "gt",
          "value": 1000
        }
      ]
    }
  ],
  "type": "filters"
}
```

##### Where region equals Nederland or Duitsland and price is greater than 1000 
```json
{
  "filters": [
    {
      "regio": [
        {
          "type": "eq",
          "value": "Nederland"
        }
      ],
      "price": [
        { 
          "type": "gt",
          "value": 1000
        }
      ]
    },
    {
      "regio": [
        {
          "type": "eq",
          "value": "Duitsland"
        }
      ],
      "price": [
        { 
          "type": "gt",
          "value": 1000
        }
      ]
     }
  ],
  "type": "filters"
}
```
##### Where region equals Nederland and price is greater than 1000 and less than the market_value 
```json
{
  "filters": [
    {
      "regio": [
        {
          "type": "eq",
          "value": "Nederland"
        }
      ],
      "price": [
        { 
          "type": "gt",
          "value": 1000
        },
        { 
          "type": "lt",
          "value": "$market_value"
        }
      ]
    }
  ],
  "type": "filters"
}
```

## Calculation
This stage has been created in order to be able to create new columns derived from existing columns. Usage could be appending text to text, doing basic calculations or creating categories for numerical records.

The different types of calculations are:
* `add`
* `subtract`
* `divide`
* `multiply`
* `abs`
* `avg`
* `ceil`
* `concat`
* `floor`
* `log` (Usage: [num, base])
* `ltrim`
* `rtrim`
* `trim`
* `max`
* `mod` (modulo)
* `pow` (Usage: [num, power])
* `round` (Usage: [num, decimals])
* `sum`
* `sd` (Standard Deviation)
* `switch`

#### Examples
##### Single calculation (basic_calculation = $price * 2)
```json
{
  "type": "calculation",
  "fields": {
    "basic_calculation": {
      "multiply": ["$price", 2]
    }
  }
}
```

##### Multilayered (multi_calculation = (price + abs(-10)) / 4)
```json
{
  "type": "calculation",
  "fields": {
    "multi_calculation": {
      "divide": [{
        "add": ["$price", {
          "abs": -10
        }]
      }, 4]
    }
  }
}
```

##### Switch statement
```json
{
  "type": "calculation",
  "fields": {
    "segment": {
      "switch": [
        { 
          "if": [{ 
            "gte": ["$price", 1500]
          }],
          "then": "Upper"
        },
        { 
          "default": true,
          "then": "Lower"
        }
      ]
    }
  }
}
```

## Kpis
In order to create calculated values over all the remaining records the kpis stage exists. In this stage the kpi keys must be declared. All keys not set here will not be passed down. 

Name | Value
---|---
fields | Object

The field needs to get a name, type of calutation and the column on which to perform the calculation.
The possible options for the type of fields include:
 * `sum`
 * `avg` (Average/ median)
 * `min` (Minimum)
 * `max` (Maximum)
 * `sdp` (Standard Deviation Population)
 * `sds` (Standard Deviation Sample)
 * `first` (First value)
 * `last` (Last value)
 * `list`
 * `ulist` (Unique List)
 * `count` (Total values)
 * `ucount` (Total of unique values)
 
#### Example:
```json
{
  "type": "kpis",
  "fields": { 
      "total": {
          "type": "count",
          "column": "id"
      }
  }
}
```

## Questionnaire
To configure the relevant questionnaire for the user, you can write a query in order determine which data will be loaded in the questionnaire. If you don't confiugure a query please be aware it will always parse in data if the collection has data. Making a questionnaire user specific, or other condition use the query options in order to fulfill your goal. 

## Explanation
In several places of the widget configuration you have the possibility to provide your own text using a WYSIWYG. It is possible to parse in data from the query into your text. In every case only the first item of the query result is accessible. Let's say you want to parse in the address in your text, you need to do the following:

#### Example:
```js
{{templateData.key}}
```
It is also possible to parse in user data. The items which can be shown are:
 * `email`
 * `first_name` (Average/ median)
 * `last_name` (Minimum)

You can use this data in the WYSIWYG:

#### Example:
```js{
{{user.key}}
}```


# Query
The query editor is meant to create a pipeline that transforms data at the latest possible stage before presentation. This means it is intended for selecting only the data necessary for creating a widget. Some types have the query disabled as the widget doesn't need any input data. Only the developer needs rights to certain data in order to put it in the pipeline. The data will not be user specific and therefore not adhere to any set permissions.
# Pipeline
The top layer of the query editor consists of the stages together forming the pipeline. These stages can be used in any quantity and order, but caution is advised for the sake of time-to-live of the widget. At the start of the pipeline the data will be passed down of the collection selected in the form. The data for the last stage will be passed down to the widget and can then be configured in the widget configuration. The last stage will always be limited to 100 items maximum to guarantee performance. The stages can have multiple different types:
*	`left join`
*	`convert`
*	`show`
*	`rename`
*	`limit`
*	`order`
*	`group`
*	`filters`
*	`calculation`
*	`kpis`
*	`Union with`
*	`unwind`
*	`array to object`
*	`object to array`
*	`add field`
*	`toEpoch`
*	`toDate`

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

The "left join" stage also accepts "$USER_TABLE" as a collection value. When this identifier is used to system user replaces the specified left_key. The right_key must then specify whether the left_key is an ID (integer) or string For example, the following dataset:
```json
[{
  "location": "work",
  ...
  "user_id": 1,
  "user_email": "j.doe@clappform.com"
}]
```
With this query:
```json
[{
  "type": "left join",
  "collection": "$USER_TABLE",
  "left_key": "user_id", // alt: "left_key": "user_email"
  "right_key": "id" // alt: "right_key": "email"
}]
```

Returns:
```json
[{
  "location": "work",
  ...
  "user_id": {
    "name": "john",
    "lastname": "doe",
    "email": "j.doe@clappform.com"
  },
  "user_email": "j.doe@clappform.com"
}]
```

## Convert
Converts a value to a specified type.

Name | Value
--- | ---
input | Text
to |	Text
onError |	Text (Optional)
onNull |	Text (Optional)


The 'to' argument can be any valid expression that resolves to one of the following string identifiers:
*	`double`
*	`string`
*	`objectId`
*	`bool`
*	`date`
*	`int`
*	`long`
*	`decimal`

#### Example:

```json
{
  "type": "convert",
  "input": "timestamp",
  "to": "string",
  "onError": "na",
  "onNull": "null"
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
The order stage allows you to order all passed items on multiple fields. The fields object stores two arrays that contain the sorting methods. Each array contains a list of keys that the query will sort on. For example, if the query only needs to sort ascending by ID, then the stage will look like this:

```json
{
  "type": "order",
  "fields": {
    "ASC": ["id"]
  }
}
```

Name | Value
---|---
fields	| Text (Object containing two arrays)

#### Example:

```json
{
  "type": "order",
  "fields": {
      "ASC": ["id", "province"],
      "DESC": ["municipality"]
  }
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

## Union With
Performs a union of two collections; i.e. 'union with' combines results from two collections into a single result set. 


Name | Value
---|---
collection	| Text (Collection slug)
 
#### Example:
```json
{
  "type": "unionWith",
  "collection": "feedback_resultaten"
}
```
This could result in duplicates, to remove duplicates you can use the 'group' stage type.

#### Example:
```json

{
  "type": "unionWith",
  "collection": "collection1"
},
{
"type": "group",
"by": ["email"],
"fields": {
    "email": {
        "type": "first",
        "column": "email"
    }
  }
}

```

## Unwind
Deconstructs an array field from the input documents to output a document for each element. Each output document is the input document with the value of the array field replaced by the element.


Name | Value
---|---
fields	| Array (List containing names of the keys)
 
#### Example:
```json
{
  "type": "unwind",
  "fields": ["feedback_resultaten"]
}
```

## Array to object
Converts an array into a single document;


Name | Value
---|---
fields	| Object(Where the key is the new name and the value is the old name)
 
#### Example:
```json
{
  "type": "arrayToObject",
  "fields": {
    "new": "old"
  }
}
```

## Object to array
Converts a object to an array. The return array contains an element for each field/value pair in the original document. 


Name | Value
---|---
fields	| Object(Where the key is the new name and the value is the old name)
 
#### Example:
```json
{
  "type": "objectToArray",
  "fields": {
    "new": "old"
  }
}
```

## Add field
Adds new fields to object. 


Name | Value
---|---
fields	| Object(Where the key is the new name and the value is value you want to add )
 
#### Example:
```json
{
  "type": "addField",
  "fields": {
    "new": "value"
  }
}
```


## To Epoch
Converts key to epoch timestamp. This stage expects a datestring as input and returns epoch timestamp.

Name | Value
---|---
column	| Text
 
#### Example:
```json
{
  "type": "toEpoch",
  "column": "datestring" 
}
```
## To Date
Inverse of toEpoch. Converts an integer timestamp to a data string. Additionally accepts format specifiers to modify the output date string, if 'format' is not specified the stage default is "%Y-%m-%d" which outputs as "2022-06-15". You can also specify the timezone, if 'timezone' is not specified the stage default is "Europe/Amsterdam".

Name | Value
---|---
column	| Text
format	| Text (Optional)
timezone	| Text (Optional)

### Format Specifiers

Specifiers | Description | Possible Values
---|---|---
%d	| Day of Month (2 digits, zero padded) | 01-31
%G	| Year in ISO 8601 format | 0000-9999
%H	| Hour (2 digits, zero padded, 24-hour clock) | 00-23
%L	| Millisecond (3 digits, zero padded) | 000-999
%m	| Month (2 digits, zero padded) | 01-12
%M	| Minute (2 digits, zero padded) | 01-31
%S	| Second (2 digits, zero padded) | 00-60
%u	| Day of week number in ISO 8601 format (1-Monday, 7-Sunday) | 1-7
%v	| Week of Year in ISO 8601 format | 1-53
%Y	| Year (4 digits, zero padded) | 0000-9999
%z	| The timezone offset from UTC. | +/-[hh][mm]
%Z	| The minutes offset from UTC as a number. For example, if the timezone offset (+/-[hhmm]) was +0445, the minutes offset is +285. | +/-mmm
%%	| Percent Character as a Literal | %

#### Example:
```json
{
  "type": "toDate",
  "column": "datestring",
  "format": "%Y-%m-%d %H:%M:%S", // Outputs: 2022-06-15 11:06:52
  "timezone": "Europe/Amsterdam"
}
```

## Questionnaire
To configure the relevant questionnaire for the user, you can write a query in order determine which data will be loaded in the questionnaire. If you don't confiugure a query please be aware it will always parse in data if the collection has data. Making a questionnaire user specific, or other condition use the query options in order to fulfill your goal. 

## Explanation
In several places of the widget configuration you have the possibility to provide your own text using a WYSIWYG. It is possible to parse in data from the query into your text. In every case only the first item of the query result is accessible. Let's say you want to parse in the address in your text, you need to do the following:

#### Example:
{% highlight html%}
{% raw %}
{{templateData.key}}
{% endraw %}
{% endhighlight %}


It is also possible to parse in user data. The items which can be shown are:
 * `email`
 * `first_name` (Average/ median)
 * `last_name` (Minimum)

You can use this data in the WYSIWYG:

#### Example:
{% highlight html%}
{% raw %}
{{user.key}}
{% endraw %}
{% endhighlight %}

## HTML (Data Table, Single Value, List)
It is possible to customize the data table, single value widget and list overview. In order to accomplish this, you need to specify the HTML and CSS. The HTML follows the HTML and Vue JS directives. The css mist be specified by making a JSON setting. You need to configure both. If you want to use data from the query result it is possible. The data is stored inside an object. It can be accessed by using the data in your HTML

#### Example (HTML):
```html
<div :style="css.testen">
testeten
</div>
```

#### Example:
{% highlight html%}
{% raw %}
{{data.key}}
{% endraw %}
{% endhighlight %}

#### Example (CCS):
```json
{
  "testen": {
    "color": "red",
    "font-size": "40px"
  }
}
```

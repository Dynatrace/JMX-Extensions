# Ruxit JMX extensions

Every JMX extension is defined by a JSON file. The following will describe how to use those JSON files.

## Basic JSON format

A extension consists of 3 main parts: metadata, metrics and UI config. The basic format is as follows:

	{
		"version": "1.0",
		"name": "jmx.hornetq",
		"type": "JMX",
		"processTypes": [ 10, 12, 13, 16, 17, 18 ],	
		"entity": "PROCESS_GROUP_INSTANCE",
		"configUI" : {
			"displayName": "HornetQ JMX"
		},
		"metrics": [ ],
		"ui": {
			"keycharts" : [ ],
			"charts": [ ]
		}
	}
          
Every JMX extension requires the following properties:

| Field | Type | Description |
| ----- | ---- | ----------- |
| version | String | The extension version in format "d.dd", needs to be updated whenever extension definition is updated |
| name | String | A unique extension name in Java package format. Extensions developed by Ruxit start with "ruxit.", consequently one of the few restrictions is that custom estensions must not start with "ruxit.". |
| type | String | Always use "JMX"
| processTypes | Integer array | Always use [ 10, 12, 13, 16, 17, 18 ] |
| entity| String | Always use "PROCESS_GROUP_INSTANCE" |
| configUI.displayName | String | Human readable extension name. This name will be displayed on the Ruxit Monitoring extensions page when you upload it.

## Metrics
This part of the JSON defines which metrics are collected by the extension. Each metric is defined by JSON similar to the following: 

	{
		"timeseries": {
			"key": "Queue.ConsumerCount",
			"unit": "Count",
			"dimensions": [
				"rx_pid"
			]
		},
		"source": {
			"domain": "org.hornetq",
			"keyProperties": {
				"type": "Queue",
			},
			"attribute": "ConsumerCount"
		}
	}

### Timeseries

This part specifies the metadata of a metric.

| Field | Type | Description |
| ----- | ---- | ----------- |
| key | String | Metric name. Must be unique whithin this extension. |
| unit | String | Metric unit. Must be one of the Available units described below |
| dimensions | String Array | Must contain "rx_pid" at index 0. This ensures that JMX attributes get the system process id (PID) as a dimension. Further dimensions can be used to e.g. provide one metric per JMX ObjectName key property value. e.g. QueueName, ThreadPoolName, ConnectionPoolName |

Available units: 
NanoSecond, MicroSecond, MilliSecond, Second, Byte, KiloByte, MegaByte, BytePerSecond, BytePerMinute, KiloBytePerSecond, KiloBytePerMinute, MegaBytePerSecond, MegaBytePerMinute, Count, PerSecond, PerMinute

### Source

This part specifies how a metric is collected using JMX. The following attributes are required for all metrics:

| Field | Type | Description |
| ----- | ---- | ----------- |
| domain | String | Domain name of the MBean |
| keyProperties | Key, Value Pairs | Key properties of the MBean. Values can contain wildcard "*" |
| attribute | String | Name of attribute that contains the metric value |

Optional attributes are:

| Field | Type | Description |
| ----- | ---- | ----------- |
| allowAdditionalKeys | boolean | If this is false then the keyProperties need to match exactly. Additional keys in the name would lead to a mismatch. If true, then additional key properties beside those specified in "keyProperties" are allowed and ignored. |
| calculateDelta | bool | If true, calculate the change in values of the given attribute. Value = attribute(t) - attribute(t-1). This is useful for monoto |
| calculateRate | bool | If true, calculate the rate of changes per seconds. This is used in combination with calculateDelta to convert an absolute attribute (e.g. Request Count) to a rate (e.g. Requests per Second). Value = attribute / query interval
| aggregation | String | Ruxit captures a value every 10 seconds but sends one aggregate value per minute. This specifies how to aggregate these 10 second values. It is also used to aggregate multiple values if more than 1 MBean matches domain and key property filter. Possible values: SUM, AVG, MIN, MAX |
| splitting | Object | Set details below |

#### Splitting

Splittings can be used to define an additional dimension for a metric. This dimension must be defined in the "dimension" property of the timeseries and the "splitting" property of the source.


	"splitting": {
		"name": "name",
		"type": "keyProperty",
		"keyProperty": "name"
	}

The following attributes have to be present for each splitting:

| Field | Type | Description |
| ----- | ---- | ----------- |
| name | String | Must match the dimension name defined for the timeseries |
| type | String | Must always be "keyProperty" |
| keyProperty | String | Defines which key property of the ObjectName of an MBean is used for splitting. |

#### Sample for a metric with an additional splitting

The following sample shows how to define a metric that provides multiple timeseries with a single metric:

	{
		"timeseries": {
			"key": "XY.Size",
			"unit": "Count",
			"dimensions": [
				"rx_pid",
				"name"
			]
		}
		"source": {
			"domain": "com.sample",
			"keyProperties": {
				"type": "XY",
				"name": "*"
			},
			"attribute": "Size",
			"splitting": {
				"name": "name",
				"type": "keyProperty",
				"keyProperty": "name"
			}
		}
	}

E.g. with MBeans com.sample:type=XY,name=A and com.sample:type=XY,name=B this will create 2 timeseries, the first for "A" and the second for "B".

## UI

This part of the JSON defines how metrics are charted at the process page. It contains a mandatory charts section and an optional keycharts section. Each section has the same makeup and looks like this:

	{
		"keymetrics" : [
		    {
			"key" : "requestCount",
			"aggregation" : "avg",
			"mergeaggregation" : "sum",
			"displayname" : "Requests"
		    }
		],		
		"ui": {
			"keycharts" : [ ],
			"charts": [ ]
		}
	}

The keymetrics section is completely optional and allows you to define up to two metrics that should be part of the Process info graphic. It has the following attributes.

| Field | Type | Description |
| ----- | ---- | ----------- |
| key | String | he key for the time series to put into the graphic |
| aggregation | String | ??? |
| mergeaggregation | String | if the metric contains multiple dimension this defines how to aggregate multiple dimension values into a single one.|
| displayname | String | The name to display in the graphic |

Each chart section has the same makeup and looks like this:

         {
                "group": "Section Name",
                "title": "Chart Name",
                "series": [
                 {
                        "key": "MetricName",
                        "aggregation": "avg",
                        "displayname": "Display name for metric",
                        "seriestype": "area"
                 },
                 {
                        "key": "Other Metric Name",
                        "aggregation": "avg",
                        "displayname": "Display name for metric",
                        "color": "rgba(42, 182, 244, 0.6)",
                        "seriestype": "area"
                 }
                ]
         }
	

The charts section describes how to chart each metric in the details page of the process page.(Found when clicking further details.

Both section allow an array of charts to be defined. A Chart has the following required attributes:

| Field | Type | Description |
| ----- | ---- | ----------- |
| group | String | The section name that the chart should be put into |
| title | String | The name of the chart |
| series | Array | An array of timeseries and charting definitions. One chart can contain multiple metrics |

A series has the following attributes

| Field | Type | Description |
| ----- | ---- | ----------- |
| key | String | The key for the time series to chart |
| displayname | String | Display name to show for the metric |
| aggregation | String | How to aggregate multiple minute values if you look at a longer chart time frame. Possible values: SUM, AVG, MIN, MAX |
| mergeaggregation | String | Key charts do not show multiple dimensions. If the metric contains multiple dimension this defines how to aggregate multiple dimension values into a single one. |
| color | String | html notation of a color. rgb and rgba are possible. |
| seriestype | String | Chart type. Possible values are: line, area, bar |
| rightaxis | boolean | if true the metric will be put on the right instead of left axis. Ruxit does support dual axis charts. |
| stacked | boolean | if true then multiple metrics will be stacked upon each other. This only works for area and bar charts. |

## Example

See the existing extensions for samples.

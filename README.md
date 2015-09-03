# JMX plugins

Every JMX plugin is defined by a JSON file. The following will describe the format we use for those JSON files.

## Basic JSON format

A plugin consists of 3 main parts: plugin metadata, plugin metrics and plugin UI. The basic format is as follows:

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

Every JMX plugin requires the following properties:

| Field | Type | Description |
| ----- | ---- | ----------- |
| version | String | plugin version in format "d.dd", needs to be updated when plugin definition is updated |
| name | String | Unique plugin name in Java package format. Plugins developed by us start with "ruxit.". |
| type | String | Always use "JMX"
| processTypes | Integer array | Always use [ 10, 12, 13, 16, 17, 18 ] |
| entity| String | Always use "PROCESS_GROUP_INSTANCE" |
| configUI.displayName | String | Human readable plugin name

## Metrics
This part of the JSON defines which metrics are collected by the plugin. Each metric is defined by JSON similar to the following: 

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
| key | String | Metric name. Must be unique whithin this plugin. |
| unit | String | Metric unit. Must be one of "Count", ... |
| dimensions | String Array | Must contain "rx_pid" at index 0. Further dimensions can be used to e.g. provide one metric per JMX ObjectName key property value. |

Available units: 
NanoSecond, MicroSecond, MilliSecond, Second, Byte, KiloByte, MegaByte, BytePerSecond, BytePerMinute, KiloBytePerSecond, KiloBytePerMinute, MegaBytePerSecond, MegaBytePerMinute, Count, PerSecond, PerMinute

### Source

This part specifies how a metric is collected using JMX. The following attributes are required for all metrics:

| Field | Type | Description |
| ----- | ---- | ----------- |
| domain | String | Domain name of the MBean |
| keyProperties | Key, Value Pairs | Key properties of the MBean. Value can contain wildcard "*" |
| attribute | String | Name of attribute that is used to fetch metric values |

Optional attributes are:

| Field | Type | Description |
| ----- | ---- | ----------- |
| allowAdditionalKeys | bool | If true, then allow additional key properties beside those specified in "keyProperties"  |
| calculateDelta | bool | If true, calculate the change in values of the given attribute. Value = attribute(t) - attribute(t-1) |
| calculateRate | bool | If true, calculate the rate of changes per seconds. This is used in combination with calculateDelta to convert an absolute attribute (e.g. Request Count) to a rate (e.g. Requests per Second). Value = attribute / query interval
| aggregation | String | Specifies how to aggregate values if more than 1 MBean matches domain and key property filter. Possible values: SUM, AVG, MIN, MAX |
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

TODO

## Example

TODO paste e.g. some from hornetq plugin

	{
		"version": "1.0",
		"name": "jmx.hornetq",
		"type": "JMX",
		"processTypes": [ 10, 12, 13, 16, 17, 18 ]
	
		"entity": "PROCESS_GROUP_INSTANCE",
		"configUI" : {
			"displayName": "HornetQ JMX"
		},
		"metrics": [
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
			},
		],
		"ui": {

		}
	}

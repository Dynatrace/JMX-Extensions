# Dynatrace JMX extensions

JMX extensions are defined by JSON files. The following describes how to use such JSON files.

## Basic JSON format

An extension consists of 3 main elements: metadata, metrics, and UI config. The basic format is as follows:

	{
		"version": "1.0",
		"name": "custom.jmx.hornetq",
		"type": "JMX",
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
          
Each JMX extension has the following mandatory properties:

| Field | Type | Description |
| ----- | ---- | ----------- |
| version | String | The extension version in format "d.dd", must be updated whenever the extension definition is updated |
| name | String | A unique extension name in Java package format. Custom jmx plugins names should follow custom.jmx.name rule. In name  only letters, numbers and "-" ,  "_" chars are allowed for example  custom.jmx.newPlugin-Ver2 |
| type | String | Always use "JMX"
| entity| String | Always use "PROCESS_GROUP_INSTANCE" |
| configUI.displayName | String | Human readable extension name. This name is displayed on the Dynatrace Monitoring extensions page once the extension is uploaded.

## Metrics
This part of the JSON defines which metrics are collected by the extension. Each metric is defined by JSON in a format similar to the following: 

	{
		"timeseries": {
			"key": "Queue.ConsumerCount",
			"unit": "Count",
			"displayname": "Queue Consumer Count",
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
| key | String | Metric name. Must be unique whithin this extension. Only  letters, numbers and "-" ,  "_" chars are allowed.|
| unit | String | Metric unit. Must be one of the Available units described below |
| dimensions | String Array | Must contain "rx_pid" at index 0. This ensures that JMX attributes get the system process ID (PID) as a dimension. Additional dimensions can be used to, for example, provide 1 metric per JMX ObjectName key property value. For example, QueueName, ThreadPoolName, or ConnectionPoolName. Only  letters, numbers and "-" ,  "_" chars are allowed. |
| displayname | String | Metric display name represent metric in Dynatrace. This field is obligatory. Must be different than metric key.|

Available units: 
NanoSecond, MicroSecond, MilliSecond, Second, Byte, KiloByte, MegaByte, BytePerSecond, BytePerMinute, KiloBytePerSecond, KiloBytePerMinute, MegaBytePerSecond, MegaBytePerMinute, Count, PerSecond, PerMinute

### Source

This part specifies how a metric is collected using JMX. The following attributes are required for all metrics:

| Field | Type | Description |
| ----- | ---- | ----------- |
| domain | String | Domain name of the MBean |
| keyProperties | Key, Value Pairs | Key properties of the MBean. Values can contain wildcards "*" |
| attribute | String | Name of attribute that contains the metric value. |

Optional attributes are:

| Field | Type | Description |
| ----- | ---- | ----------- |
| allowAdditionalKeys | Boolean | If this is false, the keyProperties need to match exactly. Additional keys in the name will lead to a mismatch. If true, then additional key properties beside those specified in "keyProperties" are allowed and ignored. |
| calculateDelta | bool | If true, calculate the change in values of the given attribute. Value = attribute(t) - attribute(t-1). This is useful for monoto. |
| calculateRate | bool | If true, calculate the rate of changes per seconds. This is used in combination with calculateDelta to convert an absolute attribute (eg. Request Count) to a rate (eg. Requests per Second). Value = attribute / query interval
| aggregation | String | Dynatrace captures a value every 10 seconds but only sends one aggregate value per minute. This specifies how to aggregate these 10 second values. It is also used to aggregate multiple values if more than 1 MBean matches the domain and key property filter. Possible values: SUM, AVG, MIN, MAX |
| splitting | Object | Set details below |

#### Splitting

Splittings can be used to define an additional dimension for a metric. This dimension must be defined in the "dimension" property of the timeseries and the "splitting" property of the source.


	"splitting": {
		"name": "name",
		"type": "keyProperty",
		"keyProperty": "name"
	}

The following attributes must be present for each splitting:

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
			"displayname": "Queue Consumer Count",
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

For example, MBeans com.sample:type=XY,name=A and com.sample:type=XY,name=B will result in 2 timeseries, the first for "A" and the second for "B".

## UI

This part of the JSON defines how metrics are charted on the process page. It contains a mandatory charts section and an optional keycharts section. Each section has the same format and looks like this:

	{		
		"ui": {
			"keymetrics" : [
		     {
					"key" : "requestCount",
					"aggregation" : "avg",
					"mergeaggregation" : "sum",
					"displayname" : "Requests"
		     }
			],
			"keycharts" : [ ],
			"charts": [ ]
		}
	}

The keymetrics section is completely optional and allows you to define up to two metrics that should be part of the Process infographic. It has the following attributes.

| Field | Type | Description |
| ----- | ---- | ----------- |
| key | String | The key for the time series to put into the graphic. Only letters, numbers and "-" , "_" chars are allowed. |
| aggregation | String | ??? |
| mergeaggregation | String | If the metric contains multiple dimensions, this defines how to aggregate the dimension values. into a single one.|
| displayname | String | The name to display in the graphic |

Each chart section has the same format and looks like this:

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
	

The charts section describes how to chart each metric in the details section of the process page (available by clicking "Further details".

Both sections allow an array of charts to be defined. A chart has the following required attributes:

| Field | Type | Description |
| ----- | ---- | ----------- |
| group | String | The section name that the chart should be put into |
| title | String | The name of the chart |
| series | Array | An array of timeseries and charting definitions. One chart can contain multiple metrics. |

A series has the following attributes:

| Field | Type | Description |
| ----- | ---- | ----------- |
| key | String | The key for the time series to chart |
| displayname | String | Display name to show for the metric. Overwites metric displayname. Default: metric displayname.|
| aggregation | String | How multiple minute values should be aggregated in charts when viewing a longer time frame. Possible values: SUM, AVG, MIN, MAX |
| mergeaggregation | String | Key charts do not show multiple dimensions. If the metric contains multiple dimensions, this defines how to aggregate the dimension values into a single dimension. |
| color | String | HTML notation of a color (RGB or RGBA). |
| seriestype | String | Chart type. Possible values are: line, area, and bar |
| rightaxis | Boolean | If true, the metric will be placed on the right instead of the left axis. Note that Dynatrace does support dual axis charts. |
| stacked | Boolean | If true, then multiple metrics will be stacked upon each other. This only works for area and bar charts. |

## Plugin.json reference
* [Metadata](https://dynatrace.github.io/plugin-sdk/api/plugin_json_apidoc.html#metadata)
* [Metrics](https://dynatrace.github.io/plugin-sdk/api/plugin_json_apidoc.html#metrics)
* [Metrics alerts](https://dynatrace.github.io/plugin-sdk/api/plugin_json_apidoc.html#metrics-alerts)
* [Visualization](https://dynatrace.github.io/plugin-sdk/api/plugin_json_apidoc.html#visualization)
* [Plugin configuration](https://dynatrace.github.io/plugin-sdk/api/plugin_json_apidoc.html##plugin-configuration)

## Example

See the existing extensions for samples.

# Validation

The orders endpoint performs 4 layers of validation, which may lead to a full request rejection or per-item rejection. These 4 layers are:
| Order | Layer | Scope | Description |
| ------- | :------ | :------- | ------ |
| 1 | Integrity | Global |Structure related issues (ie. missing locations as referenced in orders); may also include syntax errors such as a badly formatted or invalid timestamps, unknown reserved key parameters, etc.|
| 2 | Consistency | Per-item | Covers cases were a conditionally-optional parameter should’ve been specified, requests have parameters that contradict themselves, etc.|
| 3 | Compliance | Per-item | Failure or potential future failure of the agreed business rules (ie. unfulfillable promise time, pickup-to-dropoff distance is greater than what was contractually agreed, etc.)|
| 4 | Restrictions | Per-item | Operations related restrictions (eg. an order requires temperature-controlled handling, and on courier which that restriction is available)|
###### Table 1. Validation Layers

Depending on the layer and the scope, the orders request may be rejected completely or partially. For full order rejection the response will look similar to:
```json
{
	"status": "FAILED",
	"msgs": [
		"[ERROR] [LOCATION_NOT_FOUND] location id 3 wasn't found at locations",
		"[ERROR] [LOCATION_NOT_FOUND] location id 4 wasn't found at locations"
	],
	"timestamp": 1580239556000,
	"data": null
}
```
Note in this example that since there was a layer 1 error, per-item validation wasn’t performed and thus per-item errors may also be present. For layers 2 - 4 per-item errors, the response may look similar to:
```json
{
	"status": "SUCCESSFUL",
	"msgs": [
		"[INFO] 3 orders were successfully parsed",
		"[WARNING] [ORDER_REJECTED] [A0002] order A0002 was rejected"
	],
	"timestamp": 1580239556000,
	"data": [
		{
			"status": "ACCEPTED",
			"msgs": [],
			"client_order_id": "A0001"
		},
		{
			"status": "REJECTED",
			"msgs": [
				"[ERROR] [MAX_DISTANCE_EXCEEDED] [A0002] pickup-dropoff distance is greater than 5000m"
			],
			"client_order_id": "A0002"
		},
		{
			"status": "ACCEPTED",
			"msgs": [
				"[WARNING] [NO_PICKING_LOCATION] [A0003] no picking location specified"
			],
			"client_order_id": "A0003"
		}
	]
}
```

  

Per-item errors will be reported as error on the data msgs list, and as warning on the request msgs list. Labels within brackets are reported on all messages, the possible values are DEBUG*, INFO, WARNING and ERROR (* - meant for Cargamos). This bracket labels are meant for automation, and the available values are specified on a separate document. 

The required verbosity may be specified using the verbose parameter at the call. Also note that validation is performed in the order specified at table 1. 

Finally, if the simulate parameter at the call is specified and is true, the request won’t propagate, but all the messages will be included in the response, this is meant for development purposes.

# MyEnergi-App-Api
## Investigation of the MyEnergi App 

Myenergi have released a mobile app to view and control their Zappi and Eddi products.  A public API was not released.

This repository will be used to document my investigation into the API the app is using, so that the data and calls may be used in my own Home Automation system.

With thanks to members of the myenergi.info forum:
  * sashton
  * fintan.farrell
  * MilesB

who have contributed updates, and to the folks at MyEnergi who haven't officialy sanctioned this investigation but havent asked us to stop either.

## Tools Used
  * Myenergi iOS app  (http://myenergi.info)
  * Charles Proxy (https://www.charlesproxy.com)

Charles Proxy is used as an SSL proxy between the app and the Myenergi server endpoint.

## Findings

The app makes https requests to the myenergi.net host using Digest Authentication (https://en.wikipedia.org/wiki/Digest_access_authentication) using the qop directive as "auth"

An initial request is made to https://myenergi.net/cgi-jstatus-E the server responds with a status 401 Unauthorized and requests authentication returning the realm: "MyEnergi Telemetry", qop: "auth", an initial nonce, a Stale flag, and algorithm: "MD5"

To authenticate, pass the Myenergi hub's serial number as the username, and the password setup in the app as the password.

A json response to the inital request to https://myenergi.net/cgi-jstatus-E can be obtained through a browser using the hub serial number and app password when requested for credentials.

A simple curl command will do the same:

`curl --digest -u **HUB**:**PASSWORD** -H 'accept: application/json' -H 'content-type: application/json'  --compressed 'https://myenergi.net/cgi-jstatus-E'`


## Status Messages

https://myenergi.net/cgi-jstatus-E

The server responds with a json object:

```json
{
	"eddi": [{
		"dat": "07-06-2019",		//date
		"tim": "07:28:45",		//time
		"div": 928,			//Diversion amount Watts (does not appear if zero)
		"ectp1": -7,			//physical CT connection 1 value
		"ectp2": 6,			//physical CT connection 2 value
		"ectt1": "Grid",		//CT 1 name
		"ectt2": "Generation",		//CT 2 name
		"frq": 50.07,			//Supply Frequency
		"gen": 2054,			//Generated Watts
		"grd": 969,			//Watts from Grid?
		"hno": 1,
		"pha": 3,			//phase?
		"sno": 10088888,      	//Changed Eddi Serial Number
		"sta": 3,			
		"vol": 4.1,
		"ht1": "Tank 1",		//Heater 1 name
		"ht2": "Tank 2",		//Heater 2 name
		"tp1": -1,
		"tp2": -1,
		"pri": 2,			//priority>
		"cmt": 254,
		"r1a": 1,
		"r2a": 1,
		"r2b": 1,
		"che": 1			//charge added in KWH
	}]
}
```

This gives us the basic data used on the app's main screen.   The app also makes calls to 

  * https://myenergi.net/cgi-jstatus-Z  (For Zappi data)
  
```json
{
	"zappi": [{
		"dat": "07-06-2019",		//Date
		"tim": "07:28:46",		//Time
		"div": 1376,			//Diversion amount Watts (does not appear if zero)
		"ectp1": 920,			//Physical CT connection 1 value Watts
		"ectp2": 2143,			//Physical CT connection 2 value Watts
		"ectt1": "Grid",		//CT 1 Name
		"ectt2": "Generation",		//CT 2 Name
		"frq": 49.95,			//Supply Frequency
		"gen": 2143,			//Generated Watts
		"grd": 1017,			//Watts from grid?
		"pha": 1,
		"sno": 10077777,        //Changed Zappi Serial Number
		"sta": 3,
		"vol": 244.4,			//Supply voltage
		"pri": 1,			//priority
		"cmt": 253,
		"tbh": 9,			//boost hour?
		"tbm": 15,			//boost minute?
		"tbk": 90,			//boost KWh   - Note charge remaining for boost = tbk-che
		"pst": "A",			//Status A:Disconnected, B1:Awaiting Surplus, B2:Charge Complete, C1: Transitory- unknown, C2: Charge Complete
		"mgl": 100,
		"zmo": 3,			//Zappi Mode - 1:Fast, 2:Eco, 3:Eco+
		"che": 1			//Charge added in KWh
	}]
}
```
  * https://myenergi.net/cgi-jstatus-H  (For Harvi data) - I do have a harvi, but removed it from my installation and ran cat5 to the zappi.
  
```json
{
  "harvi": [
    {
      "sno": 10077777,
      "dat": "07-06-2019",
      "tim": "12:57:11",
      "ectp1": 176,
      "ectt1": "Grid",
      "ectt2": "None",
      "ectt3": "None",
      "ect1p": 1,
      "ect2p": 1,
      "ect3p": 1
    }
  ]
}
```

In addition to these status requests, the App also makes repeated calls to 

`https://myenergi.net/cgi-set-heater-priority-E10088888  `

**Note - the last 8 digits of the request are my Eddi Serial Number - which can be found in the data returned from the call to cgi-jstatus-E**  

(I have included this in the Status section as it doesnt appear to set anything, but does return the hpri (Heater Priority?) and a cpm value)

This returns
```json
{
	"hpri": 1,
	"cpm": 15
}
```


Tapping the Zappi or Eddi icon on the main screen causes the app to call new end points relating to the appliance:

### Eddi

`https://myenergi.net/cgi-jstatus-E10088888`

```json
{
	"eddi": [{
		"dat": "07-06-2019",
		"tim": "07:34:49",
		"ectp1": 3,
		"ectt1": "Grid",
		"ectt2": "Generation",
		"frq": 50.08,
		"gen": 2656,
		"grd": 131,
		"hno": 1,
		"pha": 3,
		"sno": 10088888,    //Serial Changed
		"sta": 1,
		"vol": 4.1,
		"ht1": "Tank 1",
		"ht2": "Tank 2",
		"tp1": -1,
		"tp2": -1,
		"pri": 2,
		"cmt": 254,
		"r1a": 1,
		"r2a": 1,
		"r2b": 1,
		"che": 1
	}]
}
```
**NOTE - the Eddi response above does not include a "div" property.  This response was captured when the eddi was not diverting.  It looks like properties that have the value zero are dropped from the object. Assuming that this is the same for Zappi too. **


### Zappi

`https://myenergi.net/cgi-jstatus-Z10077777`

```json
{
	"zappi": [{
		"dat": "07-06-2019",
		"tim": "07:30:29",
		"div": 2512,
		"ectp1": 5,
		"ectp2": 3190,
		"ectt1": "Grid",
		"ectt2": "Generation",
		"frq": 49.95,
		"gen": 3125,
		"grd": 18,
		"pha": 1,
		"sno": 10077777,           //Serial Changed
		"sta": 3,
		"vol": 241.6,
		"pri": 1,
		"cmt": 253,
		"tbh": 9,
		"tbm": 15,
		"tbk": 90,
		"pst": "A",
		"mgl": 100
	}]
}
```

### Eddi Boost Time

`https://myenergi.net/cgi-boost-time-E10088888`

```json
{
	"boost_times": [{
		"slt": 11,
		"bsh": 7,
		"bsm": 30,
		"bdh": 0,
		"bdm": 0,
		"bdd": "01111100"
	}, {
		"slt": 12,
		"bsh": 18,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "01111100"
	}, {
		"slt": 13,
		"bsh": 8,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "00000011"
	}, {
		"slt": 14,
		"bsh": 19,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "00000011"
	}, {
		"slt": 21,
		"bsh": 7,
		"bsm": 30,
		"bdh": 0,
		"bdm": 0,
		"bdd": "01111100"
	}, {
		"slt": 22,
		"bsh": 18,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "01111100"
	}, {
		"slt": 23,
		"bsh": 8,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "00000011"
	}, {
		"slt": 24,
		"bsh": 19,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "00000011"
	}]
}
```
### Zappi Boost Times

`https://myenergi.net/cgi-boost-time-Z10077777`
```json
{
	"boost_times": [{
		"slt": 11,		//Slot
		"bsh": 14,		//boost start hour
		"bsm": 0,		//boost start minute
		"bdh": 0,		//boost duration hour
		"bdm": 15,		//boost duration minute
		"bdd": "01111111"	//boost days of week Monday through Sunday
	}, {
		"slt": 12,
		"bsh": 14,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "00000000"
	}, {
		"slt": 13,
		"bsh": 0,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "00000000"
	}, {
		"slt": 14,
		"bsh": 0,
		"bsm": 0,
		"bdh": 0,
		"bdm": 0,
		"bdd": "00000000"
	}]
}
```

### Historic Data

#### Minute by Minute

##### Eddi
`https://myenergi.net/cgi-jday-E10088888-2019-6-7`

**Response truncated**

```json
{
	"U10088888": [{
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 28249,
		"gen": 159,
		"v1": 2428,
		"frq": 4995,
		"nect1": 106,
		"nect2": 53
	}, {
		"min": 1,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 32160,
		"gen": 180,
		"v1": 2430,
		"frq": 4996,
		"nect1": 120,
		"nect2": 60
	}, {
		"min": 2,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 32040,
		"gen": 180,
		"v1": 2429,
		"frq": 4996,
		"nect1": 120,
		"nect2": 60
	}, {
		"min": 3,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 32280,
		"gen": 180,
		"v1": 2430,
		"frq": 4998,
		"nect1": 60,
		"nect2": 60
	}, 
  ...
```
The first object in the data array does not have a "min" property; - perhaps a bug as it looks like it is minute 0, not an overview of the whole day - this is repeated after minute 59.   Also note that after hour 0, the data objects include an "hr" property, which is not included in the first 60 elements on the array.
Data from later in the array as an example.
```json
{
		"min": 14,
		"hr": 1,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 26640,
		"gen": 180,
		"v1": 2443,
		"frq": 4999,
		"nect1": 120,
		"nect2": 60
	}
```

##### Zappi

`https://myenergi.net/cgi-jday-Z10077777-2019-6-8`

**response truncated**

```json
{
	"U10077777": [{
		"dow": "Sat",
		"dom": 8,
		"mon": 6,
		"yr": 2019,
		"imp": 42900,
		"gen": 180,
		"v1": 2448,
		"frq": 5007,
		"nect1": 42900
	}, {
		"min": 1,
		"dow": "Sat",
		"dom": 8,
		"mon": 6,
		"yr": 2019,
		"imp": 42900,
		"gen": 180,
		"v1": 2446,
		"frq": 5006,
		"nect1": 42900
	}, 
...
```

####  Hourly

##### Eddie

`https://myenergi.net/cgi-jdayhour-E10088888-2019-6-7`

```json
{
	"U10088888": [{
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 1789189,
		"gen": 11139
	}, {
		"hr": 1,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 1912708,
		"gep": 458700,
		"gen": 11501
	}, {
		"hr": 2,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 1658167,
		"gen": 12053
	}, {
		"hr": 3,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 1839060,
		"gen": 12900
	}, {
		"hr": 4,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 1319580,
		"gep": 461160,
		"gen": 300
	}, {
		"hr": 5,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 66240,
		"exp": 145680,
		"gep": 3156420,
		"h1d": 870540
	}, {
		"hr": 6,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 201120,
		"exp": 157680,
		"gep": 5845080,
		"h1d": 2622300
	}, {
		"hr": 7,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 226800,
		"exp": 287640,
		"gep": 8880060,
		"h1d": 1075920,
		"h1b": 10860
	}, {
		"hr": 8,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 198660,
		"exp": 403800,
		"gep": 7010880,
		"h1d": 2020380
	}, {
		"hr": 9,
		"dow": "Fri",
		"dom": 7,
		"mon": 6,
		"yr": 2019,
		"imp": 162480,
		"exp": 9480,
		"gep": 627540,
		"h1d": 196860,
		"h1b": 7140
	}]
}
```
Note missing hr property for first object in array

##### Zappi

`https://myenergi.net/cgi-jdayhour-Z10077777-2019-6-6`

```json
{
	"U10077777": []
}
```
No data.



## Control

### Zappi

To change the Zappi mode between Fast, Eco, Eco+  call these endpoints.   Serial numbers are included in the URL - be sure to replace 10077777 with your Zappi Serial Number

#### Fast
`https://myenergi.net/cgi-zappi-mode-Z10077777-1-0-0-0000`

#### Eco
`https://myenergi.net/cgi-zappi-mode-Z10077777-2-0-0-0000`

#### Eco+
`https://myenergi.net/cgi-zappi-mode-Z10077777-3-0-0-0000`

All requests return this
```json
{
	"status": 0,
	"statustext": ""
}
```

#### Minimum Green Level 60%
`https://myenergi.net/cgi-set-min-green-Z10077777-60`
returns
```json
{
	"mgl": 60
}
```

#### Minimum Green Level 100%
`https://myenergi.net/cgi-set-min-green-Z10077777-100`
returns
```json
{
	"mgl": 100
}
```



### Eddi

Eddi can be set to boost - Example endpoints.  Serial numbers are included in the URL - be sure to replace 10088888 with your Eddi Serial Number

#### 20 minute manual boost
`https://myenergi.net/cgi-eddi-boost-E10088888-10-1-20`

#### 60 minute manual boost
`https://myenergi.net/cgi-eddi-boost-E10088888-10-1-60`

#### Cancel boost
`https://myenergi.net/cgi-eddi-boost-E10088888-1-1-0`

All requests return this
```json
{
	"status": 0,
	"statustext": ""
}
```

## Still to come... 

  *  Understanding of response properties
  *  Zappi manual / smart / timed boosts - will need to wait for new firmware as App shows car not connected, and will not allow manipulation.
  *  Eddei timed boost
  
  



# TerraAndroid

This library allows developers to connect to all integrations provided by [Terra](https://tryterra.co) in one easy to install library!

Docs are updated to newest version: 1.4.x

For docs on older versions please see [here](https://github.com/tryterra/TerraAndroid/tree/5d57219bec7f86e272c5ef8c67da6bd1a06fe277)

## The boring stuff
This library was built on Android Studio Artic Fox with Kotlin 1.6 (jvmTarget 1.8). This package targets a minimum SDK version of 28 (Android OREO). 

## The useful yet boring stuff

This library can be broken into two parts: 
- Samsung Health, GoogleFit and FreeStyleLibre in-app integrations
- REST API integrations

### For GOOGLE FIT or SAMSUNG HEALTH

You will need to install [Health Connect](https://play.google.com/store/apps/details?id=com.google.android.apps.healthdata).

Then for Samsung Health:
Settings -> Health Connect -> Enable permissions for Samsung Health to Write/Read from Health Connect

For Google Fit:
Settings -> Health Connect ->  Enable permissions for Google Fit to Write/Read from Health Connect

You will also need to include the following in your AndroidManifest:
```
<intent-filter>
    <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
</intent-filter>
```

To the Activity you wish to show when the user checks on your privacy policy page for Health Connect. 

```
 <meta-data
     android:name="health_permissions"
     android:resource="@array/health_permissions" />
```
To the activity you wish to request permissions and use Health Connect from. 

### For FreeStyleLibre

Nothing. Really ;)

## Installation

The library is part of `mavenCentral()`! You can simply install it by adding it as a dependency in your app gradle file:

`implementation co.tryterra:terra-android:1.4.0`

## Using the library!

To connect to Samsung Health, Google Fit, or FreeStyleLibre, you will need to instantiate a `TerraManager` Class as follows:

```kotlin
Terra.instance(devId: String, referenceId: String?, context: Context, completion: (TerraManager, TerraError?) -> Unit)
```

**Arguments**
- `devId: String` ➡ The developer identifier given to your after you signed up on [Terra](https://dashboard.tryterra.co)
- `context: Context` ➡ Any application/activity context that ideally superclasses or is an instance of `android.app.Activity`
- `referenceId: String` ➡ A unique identifer for you to use in order to map the Terra User ID to your own. Defaults to null
- `completion: (TerraManager, TerraError?) -> Unit` ➡ A callback function that is called after the TerraManager instance has been initialised. You should retrieve the instance from here. If an error has occurred, an instance of TerraError will also be returned

**You will need to initialise this class everytime your app comes up from terminated or stopped state. It sets up all the previous connections that has been initialised**

## TerraManager Instance Methods
Using this class, you may now initiate any connections you wish under the `Connections` enum:

```kotlin
initConnection(connection: Connections, token: String, context: Context, customPermissions: Set<CustomPermissions>, schedulerOn: Boolean, startIntent: String?, callback: (Boolean) -> Unit)
```

- `connection: Connections` ➡ A `Connection` enum from `co.tryterra.terra.Connections`. This signifies the connection you wish to make through Terra. There are currently 3 connections you could make: FREESTYLE_LIBRE, SAMSUNG, and GOOGLE_FIT
- `token: String` ➡ A token for authentication. Generate one using the endpoint: https://docs.tryterra.co/reference/generate-authentication-token
- `customPermissions: Set<CustomPermissions>` - A set of custom permissions from `co.tryterra.terra.enums.CustomPermission`. This will affect what is requested from the integration you choose to connect for. 
- `schedulerOn: Boolean` ➡ Boolean controls whether to turn on the scheduler or not.
- `startIntent: String?` ➡ Optional (defaults to `null`) String. It signifies the Activity for which you want to start after a FreeStyleLibre Sensor scan is complete. For example if your package name is (in your `AndroidManifest.xml`) is `co.tryterra.terrademo`, and the activity you wish to start after the scan is complete is called `MainActivity`, then you would insert: `co.tryterra.terrademo.MainActivity`. **N.B This functionality only works if your Intent extends from Activity**
- `callback: (Boolean, TerraError?) -> Unit` ➡ A callback function dictating whether the initialisation was successful **RECOMMEND TO WAIT FOR THIS FUNCTION BEFORE PROCEEDING**

**N.B Running this function automatically brings up permission and login screens! You only need to execute this once per connection** 

You may then check the Terra `user_id`'s with the following function:

```kotlin
getUserId(Connections): String?
```

### Subscriptions (new in 1.2.0)

**N.B Currently only STEPS are allowed for version 1.3.0**
You can now subscribe for a specific type of data! First, you will have to set a update handler (ideally before any processing happens or before you initialise Terra). 

```kotlin
Terra.updateHandler = {dType: DataTypes, update: Update -> 
  // Do processing with data here
}
```

`Update` takes the form:

```kotlin

data class Update(
    var lastUpdated: Instant,
    var samples: List<TerraData>
)

data class TerraData(
    var value: Double,
    var timestamp: Instant
)

```

After doing so, you will simply have to run (once):

```kotin
terra.subscribe(forDataTypes: Set<DataTypes>)
```

DataTypes can be:
STEPS
HEART_RATE 
HEART_RATE_VARIABILITY
CALORIES
DISTANCE

The idea here is that anytime a new update for the data type is detected, your update handler will be called! The next time your app opens, it will call your update handler with the data that has been missed between the time your app terminated to your app reopening.

---
### Body Data

```kotlin
getBody(type: Connections, startDate: Date, endDate: Date, toWebhook: Boolean, completion: (Boolean, TerraBodyDataPayload?, TerraError?) -> Unit
```

### Sleep Data

```kotlin
getSleep(type: Connections, startDate: Date, endDate: Date, toWebhook: Boolean, completion: (Boolean, TerraSleepDataPayload?, TerraError?) -> Unit
```

### Daily Data

```kotlin
getDaily(type: Connections, startDate: Date, endDate: Date, toWebhook: Boolean, completion: (Boolean, TerraDailyDataPayload?, TerraError?) -> Unit
```

### Activity Data

```kotlin
getActivity(type: Connections, startDate: Date, endDate: Date, toWebhook: Boolean, completion: (Boolean, TerraActivityDataPayload?, TerraError?) -> Unit
```

### Nutrition Data

```kotlin
(type: Connections, startDate: Date, endDate: Date, toWebhook: Boolean, completion: (Boolean, TerraNutritionDataPayload?, TerraError?) -> Unit
```

The `startDate` and `endDate` parameters here can take both `Date` and `Long` arguments! (`Long` for UNIX Timestamp since 1st Jan 1970)

The completion will be called with 3 arguments:
- Boolean -> If the request was successful or not. If not, a TerraError instance will also be called
- TerraDataPayload -> A payload for each data type. If `toWebhook` is set to true, this returns a class with a property `reference` referring to the payload reference sent to your webhook. If `toWebhook` is set to false, then this returns the entire Terra normalised payload. 
- TerraError -> Returned if any error occurred while retrieving data

### Disconnect
We recommend performing disconnects from your backend! Simply calling our deauthentication endpoint with the user Id from `getUserId(connection: Connections)`. However we still let you do disconnect within SDK (mainly for testing purposes):

```kotlin
Terra.disconnect(userId: String, devId: String, xAPIKey: String)
```


### FreeStyleLibre Specifics

**Make sure you initialised a FreestyleLibre Connection!!**

For FreeStyleLibre, you simply have to scan your sensor and the data will be sent directly to your webhook! Please note you will need to wait for 2 vibrations with the second one being much stronger and lasts 1 second!

**A scan will try to activate the sensor immediately if it's not active! *

We also provide a callback function for data:

```kotlin
readGlucoseData(completion: (FSLSensorDetails?) -> Unit)
```

This function polls the phone till a reading is done. There is a default timeout of 60 seconds. If no scan happens in 60 seconds, the completion callback will callback with `null`.

Otherwise, it will complete with `FSLSensorDetails?` with the following structure:

```kotlin
data class FSLSensorDetails(
    val sensor_state: String,
    val status: String,
    var serial_number: String = "",
    var data: TerraGlucoseData = TerraGlucoseData()
)
```

`TerraGlucoseData` follows the same structure as shown in our [glucose data field from our models](https://docs.tryterra.co/reference/v2#body)

## Connection to REST API

This package also allows you to call the "authenticateUser" and "deauthenticateUser" endpoint along with the data request endpoints provided by [Terra API](https://docs.tryterra.co). 

The `TerraClient` class deals with all API requests in this library.

```kotlin
val terraClient: TerraClient = TerraAuthClient("YOUR X API KEY", "YOUR DEV ID")
```


### Authenticate User and Deauthenticate User

```kotlin
terraClient.authenticateUser(resource: Resource){it ->
    //Do something with the response
        Log.i(TAG, it.toString())
    }
```
Call back returns an `AuthenticateUser` datamodel with the following fields:

```kotlin
data class AuthenticateUser(
    var status: String,
    var auth_url: String,
    var user_id: String,
)
```

Similarly for deauthentication:

```kotlin
terraClient.deauthenticateUser(userId: String)
```

### Data Endpoints

The following example requests Body Data from our API with startDate and endDate parameters set as UNIX Timestamps in seconds. You may set "toWebhook" to false if you wish for the callback function to return the data payload.

```kotlin
    terra.getBody(
        startDate = Date.from(Instant.ofEpochMilli(startTime)).toInstant().epochSecond,
        endDate = Date.from(Instant.ofEpochMilli(endTime)).toInstant().epochSecond,
        toWebhook = false
    ){bodyData ->
        // Do something with body data
        Log.i(TAG, bodyData.toString())
    }
```

The other data functions would be : `getDaily`, `getActivity`, `getSleep`, `getNutrition`. These functions uses the same argument as the `getBody` function and all of them returns a callback with their own datamodels. The `toWebhook` argument defaults to `true` if not provided.

The `startDate` and `endDate` parameters here can take both `Date` and `Long` arguments! (`Long` for UNIX Timestamp since 1st Jan 1970)

The data models correspond to having fields (exactly) the same as we the ones given by our [Data Models](https://docs.tryterra.co/data-models-mark-ii)

Finally theres also `getAthlete` for which only accepts `toWebhook` argument.

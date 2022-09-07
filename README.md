# TerraAndroid

This library allows developers to connect to all integrations provided by [Terra](https://tryterra.co) in one easy to install library!

Docs are updated to newest version: 1.2.0

For docs on older versions please see [here](https://github.com/tryterra/TerraAndroid/blob/79b2c265cfc6292d52cb96dc36e1a3438200471c/README.md)

## Quickstart

A demo is given here to get you started quickly!

https://github.com/tryterra/TerraAndroidDemo

## The boring stuff
This library was built on Android Studio Artic Fox with Kotlin 1.6 (jvmTarget 1.8). This package targets a minimum SDK version of 26 (Android OREO). 

## The useful yet boring stuff

This library can be broken into two parts: 
- Samsung Health, GoogleFit and FreeStyleLibre in-app integrations
- REST API integrations

### For Samsung Health

Please make sure the users have downloaded [Health Platform](https://play.google.com/store/apps/details?id=com.samsung.android.service.health&hl=en_GB&gl=US) on their devices and linked their Samsung Health Account to Health Platform. This can be done by going on Samsung Health -> Profile -> Settings -> Connected Services -> Health Platform and giving Health Platform access to their data. 

**N.B Samsung Health Feature is only available for SELECT Samsung Devices (Android OREO)+**

### For Google Fit

For Google Fit, please register your project package name from your `AndroidManifest.xml` file with [Google API](https://console.cloud.google.com). You will need to create a new project and a new set of Oauth credentials within this page. When creating the project, you will also need to consider scopes needed for the project. Enable Fitness API and People API, and select the following scopes:

**People API**
- openid
- /auth/user.birthday.read
- /auth/user.emails.read
- /auth/user.gender.read
- /auth/userinfo.email
- /auth/userinfo.profile

**Fitness API**
- /auth/fitness.activity.read
- /auth/fitness.blood_glucose.read
- /auth/fitness.blood_pressure.read
- /auth/fitness.body.read
- /auth/fitness/heart_rate.read
- /auth/fitness.body_temperature.read
- /auth/fitness.location.read
- /auth/fitness.nutrition.read
- /auth/fitness.oxygen_saturation.read
- /auth/fitness.reproductive_health.read
- /auth/fitness.sleep.read

Finish creating the OAuth Credentials as required and you should be able to start developing with Google Fit!

### For FreeStyleLibre

Nothing. Really ;)

## Installation

The library is part of `mavenCentral()`! You can simply install it by adding it as a dependency in your app gradle file:

`implementation co.tryterra:terra-android:1.1.0`

## Using the library!

To connect to Samsung Health, Google Fit, or FreeStyleLibre, you will need to instantiate a Terra Class as follows:

```kotlin
val terra: Terra = terra = Terra(
                devId = String,
                context = Context,
                referenceId = String
                completion = (Boolean) -> Unit
            )
```

**Arguments**
- `devId: String` ➡ The developer identifier given to your after you signed up on [Terra](https://dashboard.tryterra.co)
- `context: Context` ➡ Any application/activity context that ideally superclasses or is an instance of `android.app.Activity`
- `referenceId: String` ➡ A unique identifer for you to use in order to map the Terra User ID to your own. Defaults to null
- `completion: (Boolean) -> Unit` ➡ A callback function that is called after the terra instance has been initialised. 

**You will need to initialise this class everytime your app comes up from terminated or stopped state. It sets up all the previous connections that has been initialised**

**Warning: This call can throw an `TerraInvalidRequest` error! This happens when your `dev-id` is inactive/has reached user limits!**

Using this class, you may now initiate any connections you wish under the `Connections` enum:

```kotlin
terra.initConnection(connection: Connections, token: String, context: Context, customPermissions: Set<CustomPermissions>, schedulerOn: Boolean, startIntent: String?, callback: (Boolean) -> Unit)
```

- `connection: Connections` ➡ A `Connection` enum from `co.tryterra.terra.Connections`. This signifies the connection you wish to make through Terra. There are currently 3 connections you could make: FREESTYLE_LIBRE, SAMSUNG, and GOOGLE_FIT
- `token: String` ➡ A token for authentication. Generate one using the endpoint: https://docs.tryterra.co/reference/generate-authentication-token
- `customPermissions: Set<CustomPermissions>` - A set of custom permissions from `co.tryterra.terra.enums.CustomPermission`. This will affect what is requested from the integration you choose to connect for. 
- `schedulerOn: Boolean` ➡ Boolean controls whether to turn on the scheduler or not.
- `startIntent: String?` ➡ Optional (defaults to `null`) String. It signifies the Activity for which you want to start after a FreeStyleLibre Sensor scan is complete. For example if your package name is (in your `AndroidManifest.xml`) is `co.tryterra.terrademo`, and the activity you wish to start after the scan is complete is called `MainActivity`, then you would insert: `co.tryterra.terrademo.MainActivity`. **N.B This functionality only works if your Intent extends from Activity**
- `callback: (Boolean) -> Unit` ➡ A callback function dictating whether the initialisation was successful **RECOMMEND TO WAIT FOR THIS FUNCTION BEFORE PROCEEDIDNG**

**N.B Running this function automatically brings up permission and login screens! You only need to execute this once per connection** 

**Warning: This call can throw an `TerraClassNotInitiated` error! This happens you try to call this function before the `Terra` class's completion callback is called!**


You may then check the Terra `user_id`'s with the following function:

```kotlin
terra!!.getUserId(Terra.Connections)
```
## Subscriptions (new in 1.2.0)

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

This function throws: `NotAuthenticated` and `NoUpdateHandlerDetected` errors! Please handle them appropriately. 

The idea here is that anytime a new update for the data type is detected, your update handler will be called! The next time your app opens, it will call your update handler with the data that has been missed between the time your app terminated to your app reopening.

## Getting Data 

You can get data either from the scheduled intervals you set, or by manual queries.

### Body Data

```kotlin
terra.getBody(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Sleep Data

```kotlin
terra.getSleep(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Daily Data

```kotlin
terra.getDaily(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Activity Data

```kotlin
terra.getActivity(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Nutrition Data

```kotlin
terra.getNutrition(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Athlete Data

```kotlin
terra.getAthlete(type: Terra.Connections)
```

The `startDate` and `endDate` parameters here can take both `Date` and `Long` arguments! (`Long` for UNIX Timestamp since 1st Jan 1970)

These functions will automatically send the data to your webhooks. However if needed you may also catch the data from the callback function provided:

```kotlin
terra.getBody(type: Terra.Connections, startDate: Date, endDate: Date){success, payload ->
            // success -> If the function executed successfully
            // payload -> Data following our [models](https://docs.tryterra.co/data-models-mark-ii)
        }
```
### Revoke Permissions
Mainly for users to revoke permissions to your app. i.e for Samsung Health, it will pull up permissions screen and for Google Fit, it will sign the user out and revoke all scope permissions.

```kotlin
terra!.revokePermissions(type: Connections)
```

### Disconnect
We recommend performing disconnects from your backend! Simply calling our deauthentication endpoint with the user Id from `getUserId(connection: Connections)`. However we still let you do disconnect within SDK (mainly for testing purposes):

```kotlin
Terra.disconnect(userId: String, devId: String, xAPIKey: String)
```


### FreeStyleLibre Specifics

**Make sure you initialised a FreestyleLibre Connection!!**

For FreeStyleLibre, you simply have to scan your sensor and the data will be sent directly to your webhook! Please note you will need to wait for 2 vibrations with the second one being much stronger and lasts 1 second!

**A scan will try to activate the sensor immediately if it's not active! *8

We also provide a callback function for data:

```kotlin
terra.readGlucoseData(completion: (FSLSensorDetails?) -> Unit)
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



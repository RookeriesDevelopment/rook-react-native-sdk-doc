# Rook SDK

This SDK enables apps to extract and upload data from Health Connect, with this sdk you will be able to extract and upload data from health connect, [if you need check our app demo](https://github.com/RookeriesDevelopment/rook_demo_app_android_react_native_rook_sdk/tree/main)

### Content

1. [Installation](#installation)
2. [Configuration](#configuration)
3. [Usage](#usage)
   1. [useRookSyncConfiguration](#useRookSyncConfiguration)
   2. [useRookSyncPermissions](#useRookSyncPermissions)
   3. [useRookSyncSummaries](#useRookSyncSummaries)
   4. [useRookSyncEvents](#useRookSyncEvents)

### Installation

To build a project using the Rook Health Connect in React native you need to use at least react v16 and react native v65. This SDK is only available on Android this means that it won't work with iOS.

**The minimum version of android sdk is 26, the target sdk 34 and the kotlin version >= 1.8.10**

**npm**

```
npm i react-native-rook-sdk-health-connect
```

**yarn**

```
yarn add react-native-rook-sdk-health-connect
```

### Configuration

Add your client uuid in order to be authorized, follow the next example, add at the top level of your tree components the RookConnectProvider.

```jsx
import { RookSyncGate } from "react-native-android-rook-sync";

<RookSyncGate
  environment="sandbox | production"
  clientUUID="YOUR_CLIENT_UUID"
  password="YOUR_PASSWORD"
>
  <YOUR_COMPONENTS />
</RookSyncGate>;
```

Then we need to configure the android project. open the android project inside android studio. We need to modify the AndroidManifest.xml file at the top level paste the next permissions in order to access to the Health connection records.

We need to add inside your activity tag an intent filter to open Health Connect APP

```xml
<intent-filter>
  <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
</intent-filter>
```

Your `AndroidManifest.xml` file should look like this

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    ...

    <application
      ...>
      <activity
        ...>
        <intent-filter>
            ...
        </intent-filter>

        <!-- For supported versions through Android 13, create an activity to show the rationale
        of Health Connect permissions once users click the privacy policy link. -->
          <intent-filter>
              <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
          </intent-filter>
      </activity>

       <!-- For versions starting Android 14, create an activity alias to show the rationale
     of Health Connect permissions once users click the privacy policy link. -->
      <activity-alias
        android:name="ViewPermissionUsageActivity"
        android:exported="true"
        android:permission="android.permission.START_VIEW_PERMISSION_USAGE"
        android:targetActivity=".MainActivity">

        <intent-filter>
          <action android:name="android.intent.action.VIEW_PERMISSION_USAGE" />
          <category android:name="android.intent.category.HEALTH_PERMISSIONS" />
        </intent-filter>
      </activity-alias>
    </application>
</manifest>
```

#### Obfuscation

If you are using obfuscation consider the following:

In your gradle.properties (Project level) add the following to disable R8 full mode:

```properties
android.enableR8.fullMode=false
```

If you want to enable full mode add the following rules to proguard-rules.pro:

```text
# Keep generic signature of Call, Response (R8 full mode strips signatures from non-kept items).
-keep,allowobfuscation,allowshrinking interface retrofit2.Call
-keep,allowobfuscation,allowshrinking class retrofit2.Response

# With R8 full mode generic signatures are stripped for classes that are not
# kept. Suspend functions are wrapped in continuations where the type argument
# is used.
-keep,allowobfuscation,allowshrinking class kotlin.coroutines.Continuation
```

## Usage

### useRookSyncConfiguration<a id="useRookSyncConfiguration"></a>

This hook will help you to configure the user ID you want to sync.

The updateUserID should be called as a part of your **_Log in_** flow, when your users log out from your app call clearUserID (this is optional as any call to updateUserID will override the previous userID).

```js
const useRookSyncConfiguration = () => {
  ready: boolean;
  getUserID: () => Promise<string>;
  updateUserID: (userID: string) => Promise<boolean>;
  clearUserID: () => Promise<boolean>;
  syncUserTimeZone: () => Promise<boolean>;
}
```

- `ready`: Indicates when the hook is ready to work.
- `getUserID`: Return the current user ID.
- `updateUserID`: Change the current user ID.
- `clearUserID`: Clear the current user ID.
- `syncUserTimeZone`: Update the time zone of the user, you should only call this when is strictly necessary.

#### Example

```tsx
import React, { useState } from "react";
import { View, Text, TextInput, Button, ToastAndroid } from "react-native";
import { useRookSyncConfiguration } from "react-native-android-rook-sync";

export const ConfigurationScreen = () => {
  const [userId, setUserId] = useState("User id"); // Valor inicial del UserID

  const { getUserID, updateUserID, syncUserTimeZone } =
    useRookSyncConfiguration();

  const handleUpdateUserId = async (): Promise<void> => {
    try {
      await updateUserID(userId);
      ToastAndroid.show("User updated", ToastAndroid.LONG);
    } catch (error) {
      console.log(error);
    }
  };

  const handleGetUserId = async (): Promise<void> => {
    try {
      const result = await getUserID();
      console.log(result);
      setUserId(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleTimezone = async (): Promise<void> => {
    try {
      await syncUserTimeZone();
      ToastAndroid.show("Timezone synced", ToastAndroid.LONG);
    } catch (error) {
      console.log(error);
    }
  };

  return (
    <View>
      <Text>User ID:</Text>
      <TextInput
        value={userId}
        onChangeText={(text) => setUserId(text)}
        placeholder="Ingrese el UserID"
      />
      <Button title="Actualizar UserID" onPress={handleUpdateUserId} />
      <Button title="Obtener UserID" onPress={handleGetUserId} />
      <Button title="Sync Timezone" onPress={handleTimezone} />
    </View>
  );
};
```

### useRookSyncPermissions<a id="useRookSyncPermissions"></a>

This hook will help you to request permissions to extract data. Before proceeding further, you need to ensure the user’s device is compatible with
Health Connect and check if the [APK](https://play.google.com/store/apps/details?id=com.google.android.apps.healthdata) is installed.

```tsx
export type PermissionType = "SLEEP" | "PHYSICAL" | "BODY" | "ALL";

const useRookSyncPermissions: () => {
  ready: boolean;
  checkAvailability: () => Promise<
    "INSTALLED" | "NOT_INSTALLED" | "NOT_SUPPORTED"
  >;
  openHealthConnectSettings: () => Promise<void>;
  checkPermissions: (permission: PermissionType) => Promise<void>;
  requestPermissions: (permission: PermissionType) => Promise<void>;
};
```

- `ready`: Indicates when the hook is ready to work.
- `checkAvailability`: Check if the health connect service is available.
- `openHealthConnectSettings`: Open the health connect settings.
- `checkPermissions`: Check if you have the permissions.
- `requestPermissions`: Request the permissions.

#### Example

```tsx
/* eslint-disable react-hooks/exhaustive-deps */
import React, { useEffect } from "react";
import { Button, Text, View } from "react-native";
import {
  useRookSyncConfiguration,
  useRookSyncPermissions,
} from "react-native-rook-sdk-health-connect";

export const PermissionsScreen = () => {
  const {
    ready,
    checkAvailability,
    openHealthConnectSettings,
    checkPermissions,
    requestPermissions,
  } = useRookSyncPermissions();

  const { updateUserID } = useRookSyncConfiguration();

  useEffect(() => {
    if (ready) updateUserID("9808762");
  }, [ready]);

  const handleAvailability = async (): Promise<void> => {
    try {
      const result = await checkAvailability();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleOpenHealthConnect = async (): Promise<void> => {
    try {
      const result = await openHealthConnectSettings();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleHasAllPermissions = async (): Promise<void> => {
    try {
      const result = await checkPermissions("ALL");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestAllPermissions = async (): Promise<void> => {
    try {
      const result = await requestPermissions("ALL");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleHasSleepPermissions = async (): Promise<void> => {
    try {
      const result = await checkPermissions("SLEEP");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestSleepPermissions = async (): Promise<void> => {
    try {
      const result = await requestPermissions("SLEEP");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleHasPhysicalPermissions = async (): Promise<void> => {
    try {
      const result = await checkPermissions("PHYSICAL");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestPhysicalPermissions = async (): Promise<void> => {
    try {
      const result = await requestPermissions("PHYSICAL");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleHasBodyPermissions = async (): Promise<void> => {
    try {
      const result = await checkPermissions("BODY");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestBodyPermissions = async (): Promise<void> => {
    try {
      const result = await requestPermissions("BODY");
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  return ready ? (
    <View>
      <Text>Permissions</Text>
      <Button title="Check Availability" onPress={handleAvailability} />
      <Button title="Open Health Connect" onPress={handleOpenHealthConnect} />
      <Button title="Has all Permissions" onPress={handleHasAllPermissions} />
      <Button
        title="Has sleep Permissions"
        onPress={handleHasSleepPermissions}
      />
      <Button
        title="Request sleep Permissions"
        onPress={handleRequestSleepPermissions}
      />
      <Button
        title="Has Physical Permissions"
        onPress={handleHasPhysicalPermissions}
      />
      <Button
        title="Request Physical Permissions"
        onPress={handleRequestPhysicalPermissions}
      />
      <Button title="Has Body Permissions" onPress={handleHasBodyPermissions} />
      <Button
        title="Request Body Permissions"
        onPress={handleRequestBodyPermissions}
      />
      <Button
        title="Request Permissions"
        onPress={handleRequestAllPermissions}
      />
    </View>
  ) : (
    <Text>Loading . . .</Text>
  );
};
```

### useRookSyncSummaries<a id="useRookSyncSummaries"></a>

This hook will help you to extract and send data from health connect to Rook servers.

```tsx
const useRookSyncSummaries: () => {
  ready: boolean;
  shouldSyncSleepSummariesFor: (date: string) => Promise<boolean>;
  syncSleepSummary: (date: string) => Promise<boolean>;
  shouldSyncBodySummariesFor: (date: string) => Promise<boolean>;
  syncBodySummary: (date: string) => Promise<boolean>;
  shouldSyncPhysicalSummariesFor: (date: string) => Promise<boolean>;
  syncPhysicalSummary: (date: string) => Promise<boolean>;
  syncPendingSummaries: () => Promise<boolean>;
};
```

- `ready`: Indicates when the hook is ready to work.
- `shouldSyncSleepSummariesFor`: Verify if you have summaries to sync in a specific date.
- `syncSleepSummary`: Send the summary to rook servers.
- `shouldSyncBodySummariesFor`: Verify if you have summaries to sync in a specific date.
- `syncBodySummary`: Send the summary to rook servers.
- `shouldSyncPhysicalSummariesFor`: Verify if you have summaries to sync in a specific date.
- `syncPhysicalSummary`: Send the summary to rook servers.
- `syncPendingSummaries`: In case you try to sync a summary and fail, this function help to try to send again.

#### Example

```tsx
import React from "react";
import { Text, TouchableWithoutFeedback } from "react-native";
import { StyleSheet } from "react-native";
import { View } from "react-native";
import { useRookSyncSummaries } from "react-native-android-rook-sync";

export const SyncSummariesScreen = () => {
  const {
    shouldSyncSleepSummariesFor,
    syncSleepSummary,
    shouldSyncBodySummariesFor,
    syncBodySummary,
    shouldSyncPhysicalSummariesFor,
    syncPhysicalSummary,
    syncPendingSummaries,
  } = useRookSyncSummaries();

  const handleShouldSyncSleep = async (): Promise<void> => {
    try {
      const date = "2023-09-08";
      const result = await shouldSyncSleepSummariesFor(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncSleep = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncSleepSummary(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleShouldSyncBody = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      const result = await shouldSyncBodySummariesFor(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBody = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncBodySummary(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleShouldSyncPhysical = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      const result = await shouldSyncPhysicalSummariesFor(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncPhysical = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncPhysicalSummary(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSync = async (): Promise<void> => {
    try {
      console.log("loading . . .");
      const result = await syncPendingSummaries();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  return (
    <View style={styles.container}>
      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleShouldSyncSleep}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Should Sync Sleep?</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncSleep}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Sleep</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>

      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleShouldSyncBody}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Should Sync body?</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncBody}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync body</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>

      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleShouldSyncPhysical}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Should Sync Physical?</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncPhysical}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Physical</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>

      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSync}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Pending Summaries</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: "column",
  },
  row: {
    flexDirection: "row",
  },
  gridItem: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  buttonTouch: {
    backgroundColor: "#383A4E",
    paddingVertical: 10,
    paddingHorizontal: 20,
    borderRadius: 8,
    marginHorizontal: "5%",
    marginTop: 5,
  },
  buttonText: {
    color: "white",
    fontSize: 14,
    fontWeight: "bold",
    textAlign: "center",
  },
});
```

### useRookSyncEvents<a id="useRookSyncEvents"></a>

This hook will help you to extract and send data from health connect to Rook servers.

```tsx
const useRookSyncEvents: () => {
  ready: boolean;
  syncPhysicalEvents: (date: string) => Promise<boolean>;
  syncBloodGlucoseEvents: (date: string) => Promise<boolean>;
  syncBloodPressureEvents: (date: string) => Promise<boolean>;
  syncBodyMetricsEvents: (date: string) => Promise<boolean>;
  syncBodyHeartRateEvents: (date: string) => Promise<boolean>;
  syncPhysicalHeartRateEvents: (date: string) => Promise<boolean>;
  syncHydrationEvents: (date: string) => Promise<boolean>;
  syncNutritionEvents: (date: string) => Promise<boolean>;
  syncBodyOxygenationEvents: (date: string) => Promise<boolean>;
  syncPhysicalOxygenationEvents: (date: string) => Promise<boolean>;
  syncTemperatureEvents: (date: string) => Promise<boolean>;
  syncPendingEvents: () => Promise<boolean>;
};
```

- `ready`: Indicates when the hook is ready to work.
- `syncPhysicalEvents`: Send physical events for the specified date.
- `syncBloodGlucoseEvents`: Send blood glucose events for the specified date.
- `syncBloodPressureEvents`: Send blood pressure events for the specified date.
- `syncBodyMetricsEvents`: Send body metrics events for the specified date.
- `syncBodyHeartRateEvents`: Send body heart rate events for the specified date.
- `syncPhysicalHeartRateEvents`: Send physical heart rate events for the specified date.
- `syncHydrationEvents`: Send hydration rate events for the specified date.
- `syncNutritionEvents`: Send nutrition events for the specified date.
- `syncBodyOxygenationEvents`: Send body oxygenation events for the specified date.
- `syncPhysicalOxygenationEvents`: Send physical oxygenation events for the specified date.
- `syncTemperatureEvents`: Send temperature events for the specified date.
- `syncPendingEvents`: In case you try to sync an event and fail, this function help to try to send again.

#### Example

```tsx
/* eslint-disable react-hooks/exhaustive-deps */
import React, { useEffect } from "react";
import { StyleSheet, TouchableWithoutFeedback, View } from "react-native";
import { Text } from "react-native";
import {
  useRookSyncConfiguration,
  useRookSyncEvents,
} from "react-native-android-rook-sync";

export const EventsScreen = () => {
  const {
    ready,
    syncPhysicalEvents,
    syncBloodGlucoseEvents,
    syncBloodPressureEvents,
    syncBodyMetricsEvents,
    syncBodyHeartRateEvents,
    syncPhysicalHeartRateEvents,
    syncHydrationEvents,
    syncNutritionEvents,
    syncPhysicalOxygenationEvents,
    syncBodyOxygenationEvents,
    syncTemperatureEvents,
    syncPendingEvents,
  } = useRookSyncEvents();

  const { updateUserID } = useRookSyncConfiguration();

  useEffect(() => {
    if (ready) updateUserID("9808761");
  }, [ready]);

  const handleSyncPhysicalEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-12";
      console.log("loading . . .");
      const result = await syncPhysicalEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBloodGlucoseEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-11";
      console.log("loading . . .");
      const result = await syncBloodGlucoseEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBloodPressureEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-11";
      console.log("loading . . .");
      const result = await syncBloodPressureEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBodyMetricsEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-11";
      console.log("loading . . .");
      const result = await syncBodyMetricsEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBodyHeartRateEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncBodyHeartRateEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncPhysicalHeartRateEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncPhysicalHeartRateEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncNutritionEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-12";
      console.log("loading . . .");
      const result = await syncNutritionEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncHydrationEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-12";
      console.log("loading . . .");
      const result = await syncHydrationEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncPhysicalOxygenationEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-11";
      console.log("loading . . .");
      const result = await syncPhysicalOxygenationEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBodyOxygenationEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-11";
      console.log("loading . . .");
      const result = await syncBodyOxygenationEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncTemperatureEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-11";
      console.log("loading . . .");
      const result = await syncTemperatureEvents(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncEvents = async (): Promise<void> => {
    try {
      console.log("loading . . .");
      const result = await syncPendingEvents();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  return ready ? (
    <View>
      <Text>Events</Text>
      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncPhysicalEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Physical Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncBloodGlucoseEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Blood Glucose Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>
      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncBloodPressureEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Blood Glucose Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncBodyMetricsEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Body Metrics events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>
      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncBodyHeartRateEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Body Heart Rate Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncPhysicalHeartRateEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>
                Sync Physical Heart Rate Events
              </Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>
      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncNutritionEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Nutrition Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncHydrationEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Hydration Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>
      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncBodyOxygenationEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>
                Sync Body Oxygenation Events
              </Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback
            onPress={handleSyncPhysicalOxygenationEvents}
          >
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>
                Sync Physical Oxygenation Events
              </Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>
      <View style={styles.row}>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncTemperatureEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Temperature Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        <View style={styles.gridItem}>
          <TouchableWithoutFeedback onPress={handleSyncEvents}>
            <View style={styles.buttonTouch}>
              <Text style={styles.buttonText}>Sync Events</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
      </View>
    </View>
  ) : (
    <Text>Loading . . .</Text>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: "column",
  },
  row: {
    flexDirection: "row",
  },
  gridItem: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  buttonTouch: {
    backgroundColor: "#383A4E",
    paddingVertical: 10,
    paddingHorizontal: 20,
    borderRadius: 8,
    marginHorizontal: "5%",
    marginTop: 5,
  },
  buttonText: {
    color: "white",
    fontSize: 14,
    fontWeight: "bold",
    textAlign: "center",
  },
});
```

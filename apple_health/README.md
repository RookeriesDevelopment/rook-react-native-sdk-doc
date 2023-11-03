# Apple Health

This SDK enables apps to extract and upload data from Apple Health, with this sdk you will be able to extract and upload data, [if you need check our app demo](https://github.com/RookeriesDevelopment/rook_demo_app_ios_react_native_rook_sdk)

### Content

1. [Installation](#installation)
2. [Configuration](#configuration)
3. [Usage](#usage)
   1. [useRookConfiguration](#useRookConfiguration)
   2. [useRookPermissions](#useRookPermissions)
   3. [useRookSummaries](#useRookSummaries)
   4. [useRookEvents](#useRookEvents)

### Installation

To build a project using the Rook Apple Health in React native you need to use at least react v16 and react native v65. This SDK is only available on iOS this means that it won't work with Android.

**The minimum version of iOS is 13.0**

**npm**

```
npm i react-native-rook-sdk-apple-health
```

**yarn**

```
yarn add react-native-rook-sdk-apple-health
```

### Configuration

Add your client uuid in order to be authorized, follow the next example, add at the top level of your tree components the RookSyncGate.

```jsx
import { RookSyncGate } from "react-native-rook-sdk-apple-health";

<RookSyncGate
  environment="sandbox | production"
  clientUUID="YOUR_CLIENT_UUID"
  password="YOUR_PASSWORD"
>
  <YOUR_COMPONENTS />
</RookSyncGate>;
```

Then we need to add Health Kit Framework to our project in order to that please:

- Open your project in Xcode.
- Click on your project file in the Project Navigator.
- Select your target and then click on the "Build Phases" tab.
- Click on the "+" button under the "Link Binary With Libraries" section and select "HealthKit.framework" from the list.
- Select your target and then click on the "Signing Capabilities" tab.
- Click on "Add Capability" and search for "HealthKit"

Additionally add the following to the info.plist

```xml
<key>NSHealthShareUsageDescription</key>
<string>This app requires access to your health and fitness data in order to track your workouts and activity levels.</string>
<key>NSHealthUpdateUsageDescription</key>
<string>This app requires permission to write healt data to HealthKit.</string>
```

## Usage

### useRookConfiguration<a id="useRookConfiguration"></a>

This hook will help you to configure the user ID you want to sync.

```js
const useRookConfiguration = () => {
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
import { View, Text, TextInput, Button, Alert } from "react-native";
import { useRookConfiguration } from "react-native-rook-sdk-apple-health";
import { styles } from "../styles";

export const ConfigurationScreen = () => {
  const [userId, setUserId] = useState("User id");

  const { getUserID, updateUserID, clearUserID, syncUserTimezone } =
    useRookConfiguration();

  const handleUpdateUserId = async (): Promise<void> => {
    try {
      await updateUserID(userId);
      Alert.alert("Success", "User updated", [{ text: "OK" }]);
    } catch (error) {
      console.log(error);
    }
  };

  const handleGetUserId = async (): Promise<void> => {
    try {
      const result = await getUserID();
      setUserId(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleClearUserId = async (): Promise<void> => {
    try {
      await clearUserID();
      Alert.alert("Success", "User Cleared", [{ text: "OK" }]);
    } catch (error) {
      console.log(error);
    }
  };

  const handleTimezone = async (): Promise<void> => {
    try {
      await syncUserTimezone();
      Alert.alert("Success", "timezone synced", [{ text: "OK" }]);
    } catch (error) {
      console.log(error);
    }
  };

  return (
    <View>
      <Text style={styles.white}>User ID:</Text>
      <TextInput
        style={styles.white}
        value={userId}
        onChangeText={(text) => setUserId(text)}
        placeholder="Ingrese el UserID"
      />
      <Button title="Actualizar UserID" onPress={handleUpdateUserId} />
      <Button title="Obtener UserID" onPress={handleGetUserId} />
      <Button title="Clear UserID" onPress={handleClearUserId} />
      <Button title="Sync Timezone" onPress={handleTimezone} />
    </View>
  );
};
```

### useRookPermissions<a id="useRookPermissions"></a>

This hook will help you to request permissions to extract data.

```tsx
const useRookPermissions: () => {
  ready: boolean;
  requestAllPermissions: () => Promise<void>;
  requestSleepPermissions: () => Promise<void>;
  requestUserInfoPermissions: () => Promise<void>;
  requestPhysicalPermissions: () => Promise<void>;
  requestBodyPermissions: () => Promise<void>;
};
```

- `ready`: Indicates when the hook is ready to work.
- `requestAllPermissions`: Request all the permissions.
- `requestSleepPermissions`: Request sleep the permissions.
- `requestPhysicalPermissions`: Request Physical the permissions.
- `requestBodyPermissions`: Request Body the permissions.
- `requestUserInfoPermissions`: Request User Info the permissions.

#### Example

```tsx
import React from "react";
import { Button, Text, View } from "react-native";
import { useRookPermissions } from "react-native-rook-sdk-apple-health";

export const PermissionsScreen = () => {
  const {
    ready,
    requestAllPermissions,
    requestSleepPermissions,
    requestPhysicalPermissions,
    requestBodyPermissions,
    requestUserInfoPermissions,
  } = useRookPermissions();

  const handleRequestAllPermissions = async (): Promise<void> => {
    try {
      const result = await requestAllPermissions();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestSleepPermissions = async (): Promise<void> => {
    try {
      const result = await requestSleepPermissions();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestPhysicalPermissions = async (): Promise<void> => {
    try {
      const result = await requestPhysicalPermissions();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestBodyPermissions = async (): Promise<void> => {
    try {
      const result = await requestBodyPermissions();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleRequestUserInfoPermissions = async (): Promise<void> => {
    try {
      const result = await requestUserInfoPermissions();
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  return ready ? (
    <View>
      <Text>Permissions</Text>
      <Button
        title="Request sleep Permissions"
        onPress={handleRequestSleepPermissions}
      />
      <Button
        title="Request Physical Permissions"
        onPress={handleRequestPhysicalPermissions}
      />
      <Button
        title="Request Body Permissions"
        onPress={handleRequestBodyPermissions}
      />
      <Button
        title="Request User Info Permissions"
        onPress={handleRequestUserInfoPermissions}
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

### useRookSummaries<a id="useRookSummaries"></a>

This hook will help you to extract and send data from health connect to Rook servers.

```tsx
const useRookSummaries: () => {
  ready: boolean;
  syncSleepSummary: (date: string) => Promise<void>;
  syncPhysicalSummary: (date: string) => Promise<void>;
  syncBodySummary: (date: string) => Promise<void>;
  syncPendingSummaries: () => Promise<void>;
};
```

- `ready`: Indicates when the hook is ready to work.
- `syncSleepSummary`: Send the summary to rook servers.
- `syncBodySummary`: Send the summary to rook servers.
- `syncPhysicalSummary`: Send the summary to rook servers.
- `syncPendingSummaries`: In case you try to sync a summary and fail, this function help to try to send again.

#### Example

```tsx
import React from "react";
import { Text, TouchableWithoutFeedback } from "react-native";
import { StyleSheet } from "react-native";
import { View } from "react-native";
import { useRookSummaries } from "react-native-rook-sdk-apple-health";

export const SummariesScreen = () => {
  const {
    syncSleepSummary,
    syncBodySummary,
    syncPhysicalSummary,
    syncPendingSummaries,
  } = useRookSummaries();

  const handleSyncSleep = async (): Promise<void> => {
    try {
      const date = "2023-09-07";
      console.log("loading . . .");
      const result = await syncSleepSummary(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBody = async (): Promise<void> => {
    try {
      const date = "2023-09-07";
      console.log("loading . . .");
      const result = await syncBodySummary(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncPhysical = async (): Promise<void> => {
    try {
      const date = "2023-09-07";
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
    <View>
      <TouchableWithoutFeedback onPress={handleSyncSleep}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync Sleep</Text>
        </View>
      </TouchableWithoutFeedback>

      <TouchableWithoutFeedback onPress={handleSyncBody}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync body</Text>
        </View>
      </TouchableWithoutFeedback>

      <TouchableWithoutFeedback onPress={handleSyncPhysical}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync Physical</Text>
        </View>
      </TouchableWithoutFeedback>

      <TouchableWithoutFeedback onPress={handleSync}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync Pending Summaries</Text>
        </View>
      </TouchableWithoutFeedback>
    </View>
  );
};

const styles = StyleSheet.create({
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

### useRookEvents<a id="useRookEvents"></a>

This hook will help you to extract and send data from health connect to Rook servers.

```tsx
const useRookEvents: () => {
  ready: boolean;
  syncBodyHeartRateEvent: (date: string) => Promise<void>;
  syncPhysicalHeartRateEvent: (date: string) => Promise<void>;
  syncBodyOxygenationEvent: (date: string) => Promise<void>;
  syncPhysicalOxygenationEvent: (date: string) => Promise<void>;
  syncTrainingEvent: (date: string) => Promise<void>;
};
```

- `ready`: Indicates when the hook is ready to work.
- `syncBodyHeartRateEvents`: Send body heart rate events for the specified date.
- `syncPhysicalHeartRateEvents`: Send physical heart rate events for the specified date.
- `syncBodyOxygenationEvents`: Send body oxygenation events for the specified date.
- `syncPhysicalOxygenationEvents`: Send physical oxygenation events for the specified date.
- `syncTrainingEvent`: Send training events for the specified date.

#### Example

```tsx
import React from "react";
import { StyleSheet, TouchableWithoutFeedback, View } from "react-native";
import { Text } from "react-native";
import { useRookEvents } from "react-native-rook-sdk-apple-health";

export const EventsScreen = () => {
  const {
    ready,
    syncBodyHeartRateEvent,
    syncPhysicalHeartRateEvent,
    syncBodyOxygenationEvent,
    syncPhysicalOxygenationEvent,
    syncTrainingEvent,
  } = useRookEvents();

  const handleSyncBodyHeartRateEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-07";
      console.log("loading . . .");
      const result = await syncBodyHeartRateEvent(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncPhysicalHeartRateEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncPhysicalHeartRateEvent(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncPhysicalOxygenationEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncPhysicalOxygenationEvent(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleSyncBodyOxygenationEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncBodyOxygenationEvent(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  const handleTrainingEvents = async (): Promise<void> => {
    try {
      const date = "2023-09-06";
      console.log("loading . . .");
      const result = await syncTrainingEvent(date);
      console.log(result);
    } catch (error) {
      console.log(error);
    }
  };

  return ready ? (
    <View>
      <Text>Events</Text>
      <TouchableWithoutFeedback onPress={handleSyncBodyHeartRateEvents}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync Body Heart Rate Events</Text>
        </View>
      </TouchableWithoutFeedback>

      <TouchableWithoutFeedback onPress={handleSyncPhysicalHeartRateEvents}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync Physical Heart Rate Events</Text>
        </View>
      </TouchableWithoutFeedback>

      <TouchableWithoutFeedback onPress={handleSyncBodyOxygenationEvents}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync Body Oxygenation Events</Text>
        </View>
      </TouchableWithoutFeedback>

      <TouchableWithoutFeedback onPress={handleSyncPhysicalOxygenationEvents}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>
            Sync Physical Oxygenation Events
          </Text>
        </View>
      </TouchableWithoutFeedback>
      <TouchableWithoutFeedback onPress={handleTrainingEvents}>
        <View style={styles.buttonTouch}>
          <Text style={styles.buttonText}>Sync Trainings Events</Text>
        </View>
      </TouchableWithoutFeedback>
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

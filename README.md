# BlueRange SDK for Android

## What does this SDK do?
"BlueRange SDK for Android" is a library for the Android platform providing an API to develop applications that interact with Bluetooth Low Energy beacons in general and BlueRange SmartBeacons in specific. The API is intended to provide an easy to use interface to develop time and location based applications that make use of BlueRange mesh networks. For more information about the BlueRange technology, have a closer look at this project on GitHub. 

The SDK also provides a seamless integration into an infrastructure based on the Enterprise Mobility Management platform Relution. Relution SmartBeacon Management is part of Relution and allows you to control and monitor a mesh of SmartBeacons. Using Relution you can assign each beacon a message that will be send out in regular intervals. At the moment Relution supports iBeacons and Relution Tag messages. 

The idea behind Relution Tags is to assign each beacon a number of tags that the beacon will send out periodically. A mobile app using this SDK receives these tags and individually interprets them by e.g. executing an action, showing the user product information etc. Since the app itself is responsible for mapping each tag to a specific behavior, there is no need of continuous internet availability. Although the concept of Relution tags does not allow you to change the app's behavior dynamically, Relution makes it possible to exchange the tags a beacon sends out and, thus, gives your app a partial dynamic behavior without the requirement of constant internet access. 

iBeacons, on the other side, are currently used by Relution to realize a concept, which we call "campaigns". A campaign allows you to define a set of actions that will be triggered within a mobile app, whenever the device is next to a beacon that takes part in the campaign. In contrast to the concept of Relution tags, the mapping of iBeacon and app behavior is not done by the app itself but by Relution. The disadvantage of this concept is that your app constantly needs internet access. The advantage, however, is, that the content and behavior that is associated with an action, will be instantly updated, whenever it was changed in Relution.

## Features
Currently the BlueRange SDK supports Android 4.3 and iOS 8.0 devices. However, to enable advertising, Android devices must additionally support the Bluetooth LE peripheral mode and run at least on API level 21.

The BlueRange SDK is divided into two layers, a core and a service layer. The core layer consists of a set of message processing components that can be combined to a flexible message processing architecture. The service layer builds on top of the core layer and is responsible for the integration of the message processing components and the Relution platform. The current version supports the following features:

### Core
#### Advertising
- Sending advertising messages of arbitrary data.

#### Scanning
- Scanning iBeacon, Relution Tag and BlueRange specific messages. 
- The scan procedure will be continued when the app is running in background, even when the user attempts to terminate the app.
- The energy consumption can be adjusted dynamically by changing scan properties like the scan interval.

#### Logging
- Messages can be saved persistently and postprocessed at a later time. The log will be persisted with a minimum amount of energy consumption.

#### Reporting
- Messages logged over a long period of time can be transformed to status reports and periodically sent to a mesh management system like the Relution SmartBeacon management system.

#### Triggering
- A stream of beacon messages can trigger an action which will be executed based on time and location based parameters. Actions can e.g. be executed after a predefined delay or locked for a specific duration. Moreover, actions can be defined to trigger only inside a certain range to the beacon or can have a validation period.
- To make the execution of distance dependent actions more stable, the messages' received signal strength is averaged over time by using a linearly weighted moving average filter.
- Notice: the devices, we used for testing (Galaxy S4, Nexus 5, Nexus 9) were able to receive about 1-2 messages/sec. However, some devices and Android versions seem to scan very infrequently Bluetooth Low Energy advertising messages, even if the ```BluetoothLeScanner```'s scan mode is set to ```SCAN_MODE_LOW_LATENCY```. We hope that this problem will be solved in future Android versions.

### Service
As mentioned above, the service layer builds on top of the core layer and provides some high level features to integrate the message processing components with the Relution SmartBeacon Management platform. Concretely, it enables you
- to calibrate iBeacon messages to improve the precision of distance dependent actions.
- to map Relution Tags to their names and descriptions defined in the Relution platform.
- to periodically send specific advertising messages, that will be used to display a heatmap on the Relution SmartBeacon management platform.

## Documentation
- All currently supported features are bundled in the ```RelutionIoTService``` class. However, if you want more flexibility, you can build your app on top of the core and service components. To get an overview of the packages and classes, please have a look at our documentation in the ```/docs``` directory.

## Getting started
- In order to get started with your own app, you first need to unzip the "BlueRangeSDK_Android.zip".
- Next, start the Android IDE of your choise (e.g. Android Studio) and import the project contained in the unpacked folder. To do this with Android Studio, just click on "Open an existing Android project" right after Android Studio has been started and choose the unpacked folder.
- After the project has been imported, just have a look at the project structure on the left side bar.
- As you can see, the ```com.mway.bluerange.android``` package contains two subpackages ```examples``` and ```sdk```.
- The ```sdk``` package consists of the SDK's source code. So if you are interested in how the SDK components are implemented, you should have a closer look at these packages.
- In the ```examples``` package you can see how the SDK components can be used within your app. The ```systemtests``` package contains code examples that show you, how to use the SDK's core components.
- The ```nearyou``` package contains a reference application displaying you the currently received beacon messages and the executed actions. You can test the app, if you have a Bluetooth Low Energy capable device. In order to start the app, you only need to build, deploy and start the application. If you are using Android Studio, you can simply use the keyboard shortcut ```Shift+F10```.
- The best way to start writing your own app would be to replace all classes of the NearYou app and transform the code to your own needs.

## Reference application

You can start the NearYou app to see the messages that are currently being received by the app or to check whether you have correctly configured the actions. In order to calibrate an iBeacon, just place your device 1 meter away from the beacon, wait about 5 seconds, and click on the "Calibrate" button. To verify, whether the iBeacon has correctly been calibrated, the "Calibrated RSSI" value should have changed to a new value.

![alt tag](https://raw.githubusercontent.com/ehrlich89/images/master/02_iBeacons2.png)

Next to the tabs "iBeacon", "Relution Tags" and "JoinMe", the "Visited", "Content" and "Notification" tabs allow you to monitor actions, as you can see in the screenshot below. The "last update" field displays the last time, when a message was received that was able to initiate the action. "Trigger threshold" shows you the distance threshold that you specified when you defined the action inside a campaign. The bar besides "distance to trigger" shows the distance to the trigger threshold. A negative distance means that you are inside the radius defined by the trigger threshold. Finally, the "Trigger locked" and "Trigger delayed" fields indicate the lock and delay duration for this action.

![alt tag](https://raw.githubusercontent.com/ehrlich89/images/master/05_Action2.png)

## Sample code
The following section shows you, how the most important SDK features can be integrated inside your app. As described above, you can use the ```RelutionIoTService``` class, if you do not need the flexibility of the underlying message processing architecture and just want to get informed about executed actions, incoming messages or just want to turn on/off features.

### Configuration
The first thing you have to do is to add the ```RelutionIoTService``` to your ```AndroidManifest.xml```. Next, create an instance of this class inside the ```onCreate``` method of your  ```android.app.Application``` subclass. To start the service, you must call the ```startInForegroundMode``` or ```startInBackgroundMode``` method. If you start the service in background mode, the service will continue processing messages, even when the user tries to terminate the app. The minimum requirement before starting the service, however, is to call the ```setConfig``` method, where you pass the Relution URL, the organization UUID and the authentication data you use to log in. Moreover, you can enable/disable features by calling the ```set...Enabled``` methods. By default, all features are enabled. However, having all features turned on, might decrease the overall performance of your application. The easiest way to improve the performance is to set ```loggingEnabled``` to ```true```. This will stop logging debug messages to the Android logcat. ```setCampaignActionTriggerEnabled``` enables/disables the triggering mechanism which is used to implement the campaign concept. If you enable ```heatmapGeneration```, the ```RelutionIoTService``` will constantly send specific advertising messages to the beacons. The beacons will collect this information and send it to Relution, where it will be used to generate and display the heatmap. Setting ```heatmapReporting``` to ```true```, will turn a status reporter on, that constantly collects BlueRange specific beacon messages and periodically sends status reports to Relution which can be analyzed and postprocessed at a later time.

```java
String baseUrl = "https://url-to-relution.com";
String organizationUuid = "11111111-2222-3333-4444-555555555555";
String username = "your_username";
String password = "your_password";

new RelutionIoTService()
		.setConfig(baseUrl, organizationUuid, username, password)
		.setLoggingEnabled(true)
		.setCampaignActionTriggerEnabled(true)
		.setHeatmapGenerationEnabled(true)
		.setHeatmapReportingEnabled(true)
		.startInForegroundMode(context.getApplicationContext());
```

### Beacon messages
If your app must react to incoming beacon messages, you can register a ```BeaconMessageObserver``` instance by calling ```addBeaconMessageObserver```, as shown below. Take in mind, however, that your observer will only receive messages that were processed by one of the messages processing components, the ```RelutionIoTService``` uses. If you need to scan for specific beacon messages, use the ```BeaconMessageScanner``` class.
```java
RelutionIoTService.addBeaconMessageObserver(new RelutionIoTService.BeaconMessageObserver() {
	@Override
	public void onMessageReceived(BeaconMessage message) {
		// Do something with the message.
		if (message instanceof IBeaconMessage) {
			// Do something with the iBeacon message.
			IBeaconMessage iBeaconMessage = (IBeaconMessage) message;
		}
	}
});
```

### iBeacon calibration
To calibrate an iBeacon inside your app (like in the reference application), just place your device 1 meter away from the beacon
and send an averaged RSSI value of the iBeacon message to Relution by calling ```calibrateIBeacon```, as shown below:
```java
RelutionIoTService.addBeaconMessageObserver(new RelutionIoTService.BeaconMessageObserver() {
	@Override
	public void onMessageReceived(BeaconMessage message) {
		// Do something with the message.
		if (message instanceof IBeaconMessage) {
			// Get the iBeacon message.
			IBeaconMessage iBeaconMessage = (IBeaconMessage) message;
			// User moves to a place 1 meter away from the beacon that sends the iBeacon message...
			// Calibrate the iBeacon message.
			RelutionIoTService.calibrateIBeacon(iBeaconMessage.getIBeacon(), iBeaconMessage.getRssi());
		}
	}
});
```

### Beacon actions
If beacons take part in a campaign, your app should react to actions defined in the campaign. You can do this by adding observer instances to the ```RelutionIoTService``` instance for each specific type of action:
```java
RelutionIoTService.addBeaconNotificationActionObserver(new RelutionIoTService.BeaconNotificationActionObserver() {
	@Override
	public void onNotificationActionExecuted(BeaconNotificationAction notificationAction) {
		// Do something...
	}
});
RelutionIoTService.addBeaconContentActionObserver(new RelutionIoTService.BeaconContentActionObserver() {
	@Override
	public void onContentActionExecuted(BeaconContentAction contentAction) {
		// Do something...
	}
});
RelutionIoTService.addBeaconTagActionObserver(new RelutionIoTService.BeaconTagActionObserver() {
	@Override
	public void onTagActionExecuted(BeaconTagAction tagAction) {
		// Do something...
	}
});
```

### Relution tags
If you want to make use of Relution Tags, register a ```RelutionTagObserver``` to get informed about all received Relution Tags. If you need to have access to the name or description of a Relution Tag, just call ```getTagInfoForTag```:

```java
RelutionIoTService.addRelutionTagObserver(new RelutionIoTService.RelutionTagObserver() {
	@Override
	public void onTagReceived(long tag, RelutionTagMessage message) {
		try {
			RelutionIoTService.getTagInfoForTag(tag);
		} catch (RelutionTagInfoRegistry.RelutionTagInfoRegistryNoInfoFound relutionTagInfoRegistryNoInfoFound) {
			// ...
		}
	}
});
```

### Scanning
If you want to scan for specific beacon messages, you can create an instance of ```BeaconMessageScanner``` and
configure it, so that the scanner scans for these messages. Next, register a ```BeaconMessageStreamNodeReceiver``` whose ```onReceivedMessage``` will be called, whenever a matching message has been received. You can also add message types to the scanner's configuration after you have started the scanner. To do this, just call the appropriate methods of the ```BeaconMessageScannerConfig``` object.

```java
final BeaconMessageScanner beaconScanner = new BeaconMessageScanner(this);
final BeaconMessageScannerConfig config = new BeaconMessageScannerConfig(beaconScanner);
config.scanIBeacon("b9407f30-f5f8-466e-aff9-25556b57fe6d", 45, 1);
config.scanIBeacon("c9407f30-f5f8-466e-aff9-25556b57fe6d", 46, 2);
config.scanRelutionTags(new long[]{13, 2});
config.scanJoinMeMessage();
beaconScanner.setConfig(config);
beaconScanner.addReceiver(new BeaconMessageStreamNodeReceiver() {
	@Override
	public void onMeshActive(BeaconMessageStreamNode senderNode) {
		Log.d(Config.projectName, "onMeshActive");
	}

	@Override
	public void onReceivedMessage(BeaconMessageStreamNode senderNode, BeaconMessage message) {
		Log.d(Config.projectName, "onBeaconUpdate");
		Log.d(Config.projectName, message.toString());
	}

	@Override
	public void onMeshInactive(BeaconMessageStreamNode senderNode) {
		Log.d(Config.projectName, "onMeshInactive");
	}
});
beaconScanner.startScanning();
```


### Advertising
To periodically send advertising messages, just call one of the ```start``` methods of the ```BeaconAdvertiser``` class:
```java
public class AdvertisingService extends BlueRangeService {
    @Override
    public void onStarted() {
        try {
            BeaconAdvertiser advertiser = new BeaconAdvertiser(this.getApplicationContext());
            advertiser.startAdvertisingDiscoveryMessage();
        } catch (PeripheralAdvertisingNotSupportedException e) {
            Log.d("Advertiser", "This device does not support advertising in peripheral mode!");
            e.printStackTrace();
        }
    }
}
```

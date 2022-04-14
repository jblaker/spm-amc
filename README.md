# Omniture (Adobe Marketing Cloud - Adobe Analytics) Plugin for Brightcove Player SDK for iOS, version {!{ versionnumber placeholder string }!}

## Installation

The Omniture plugin is a static library framework which supports Adobe Marketing Cloud v4.21.2 and Adobe Video Heartbeat v2.3.0. The Marketing Cloud and Video Heartbeat libraries **are not** included with this SDK and must be manually added to your project. Instructions for downloading the libraries are provided below.

### CocoaPods

You can use [CocoaPods][cocoapods] to add the Omniture plugin to your project.  You can find the latest `Brightcove-Player-Omniture` podspec [here][podspecs]. To use this spec, add the following to the top of your Podfile: `source 'https://github.com/brightcove/BCOVSpecs.git'`.

#### Static Framework example:

```bash
source 'https://github.com/brightcove/BrightcoveSpecs.git'

target 'MyOmniturePlayer' do
  pod 'Brightcove-Player-Omniture'
end
```

#### XCFramework example:

XCFrameworks can be installed by appending the `/XCFramework` subspec to the pod name.

```bash
source 'https://github.com/brightcove/BrightcoveSpecs.git'

target 'MyOmniturePlayer' do
  pod 'Brightcove-Player-Omniture/XCFramework'
end
```

### Manual

To add the Omniture Plugin for Brightcove Player SDK to your project manually:

1. Install the latest version of the [Brightcove Player SDK][bcovsdkmanualsetup].
2. Download the latest zip'ed release of the plugin from the [release page][release].
1. Add the `BrightcoveAMC.framework` or `BrightcoveAMC.xcframework` project.  You can do this by right-clicking on the Frameworks folder and choose "Add Files To" option and select the `BrightcoveAMC.framework` or `BrightcoveAMC.xcframework` from the path where it is stored.
1. On the "Build Settings" tab of your application target, ensure that the "Framework Search Paths" include the path to the framework. This should have been done automatically unless the framework is stored under a different root directory than your project.
1. On the "Build Phases" tab of your application target, add the following to the "Link
    Binary With Libraries" phase:
    * `BrightcoveAMC.framework` / `BrightcoveAMC.xcframework`
1. On the "Build Settings" tab of your application target:
    * Ensure that `-ObjC` has been added to the "Other Linker Flags" build setting.
1. Download Adobe Marketing Cloud following Adobe's [instructions][adobemarketingcloud] and add `AdobeMobile.xcframework` to your project.
1. Download Adobe Video Heartbeat following Adobe's [instructions][adobevideoheartbeat] and add `MediaSDK.xcframework` to your project.
1. Add the Marketing Cloud and Video Heartbeat header folders to the Header Search Path settings of your project.
1. (**Universal Framework** only) On the "Build Phases" tab, add a "Run Script" phase with the command `bash ${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/BrightcoveAMC.framework/strip-frameworks.sh`. Check "Run script only when installing". This will remove unneeded architectures from the build, which is important for App Store submission.
1. (**Apple Silicon** only) On the "Build Settings" tab of your application target:
    * Ensure that `arm64` has been added to your "Excluded Architectures" build setting for `Any iOS Simulator SDK`.

[cocoapods]: http://cocoapods.org
[podspecs]: https://github.com/brightcove/BrightcoveSpecs/tree/master/Brightcove-Player-Omniture
[release]: https://github.com/brightcove/brightcove-player-sdk-ios-omniture/releases
[bcovsdkmanualsetup]: https://github.com/brightcove/brightcove-player-sdk-ios#ManualInstallation
[adobemarketingcloud]: https://docs.adobe.com/content/help/en/mobile-services/ios/overview.html
[adobevideoheartbeat]: https://experienceleague.adobe.com/docs/media-analytics/using/sdk-implement/download-sdks.html?lang=en

### Swift Package Manager

To add the SSAI Plugin for Brightcove Player SDK to your project with Swift Package Manager: 

1. First [follow the steps][corespm] to add the Core XCFramework with Swift Package Mananger.
1. Add the SSAI package to Swift Package Manager using `https://github.com/brightcove/brightcove-player-sdk-ios-ssai.git`.
1. Link to the following libraries:
    * libsqlite3.0.tbd
1. On the "Build Settings" tab of your application target:
    * Ensure that `-ObjC` has been added to the "Other Linker Flags" build setting.
1. Download Adobe Marketing Cloud following Adobe's [instructions][adobemarketingcloud] and add `AdobeMobile.xcframework` to your project.
1. Download Adobe Video Heartbeat following Adobe's [instructions][adobevideoheartbeat] and add `MediaSDK.xcframework` to your project.
1. Add the Marketing Cloud and Video Heartbeat header folders to the Header Search Path settings of your project.

*Note: The Open Measurement XCFramework will be automatically installed with the SSAI XCFramework.*

[corespm]: https://github.com/brightcove/brightcove-player-sdk-ios#ManualInstallation

## Quick Start

BrightcoveAMC is a bridge between the [Brightcove Player SDK for iOS][bcovsdk] and Adobe video tracking, including [Adobe Video Heartbeat v2][adobevideoheartbeat] and [Adobe Media Tracking v4][adobemobileservice]. BrightcoveAMC tracks basic video behaviors: video ready, video completed, video play, and video pause. 

## Client Implementation

To setup Adobe Video Heartbeat, Omniture Plugin clients need to implement instances of BCOVAMCVideoHeartbeatConfigurationPolicy. To setup Adobe Media Tracking, an instance of BCOVAMCMediaSettingPolicy is required. These [policies][strategypattern] allow for customization of ADBMediaHeartbeatConfig or ADBMediaSettings objects for each playback session.

[strategypattern]: https://en.wikipedia.org/wiki/Strategy_pattern

### Video Heartbeat v2

This example uses video heartbeat tracking.

```
[1] // Create the VHB configuration policy object.
    BCOVAMCVideoHeartbeatConfigurationPolicy videoHeartbeatConfigurationPolicy = ^ADBMediaHeartbeatConfig *(id<BCOVPlaybackSession> session) {

        ADBMediaHeartbeatConfig *configData = [[ADBMediaHeartbeatConfig alloc] init];
        
        configData.trackingServer = <adobe-assigned_tracking_server>";
        configData.channel = <cutomize_sample_channel>;
        configData.appVersion = <app_version>;
        configData.ovp = <online_video_platform>;
        configData.playerName = <player_name>;
        configData.ssl = <YES | NO>;

        // Set debugLogging to true to activate debug tracing. Remove it in production.
        configData.debugLogging = YES;
        
        return configData;
    };

[2] // Create the Brightcove AMC analytics policy object fromw1w the VHB configuration policy object.
    BCOVAMCAnalyticsPolicy *heartbeatPolicy = [[BCOVAMCAnalyticsPolicy alloc] initWithHeartbeatConfigurationPolicy:videoHeartbeatConfigPolicy];

    BCOVAMCSessionConsumer *sessionConsumer = [BCOVAMCSessionConsumer heartbeatAnalyticsConsumerWithPolicy:heartbeatPolicy delegate:self];
   
    BCOVPlayerSDKManager *manager = [BCOVPlayerSDKManager sharedManager];
    id<BCOVPlaybackController> controller = [manager createPlaybackController];
    controller.delegate = self;
    [self.view addSubview:controller.view];
   
[3] // Add the Brightcove AMC session consumer to the playback controller.
    [controller addSessionConsumer:sessionConsumer];       

[4] // Find and play a video.
    NSString *policyKey = <your-policy-key>;
    NSString *accountId = <your-account-id>;
    NSString *videoID = <your-video-id>;
    BCOVPlaybackService *service = [[BCOVPlaybackService alloc] initWithAccountId:accountId
                                                                        policyKey:policyKey];
    [service findVideoWithVideoID:videoID
                       parameters:nil
                       completion:^(BCOVVideo    *video,
                                    NSDictionary *jsonResponse,
                                    NSError      *error) {

        [controller setVideos:@[ video ]];
        [controller play];

    }];
```
       
1. Create the video heartbeat configuration policy block which will be called at the start of each playback session. The policy allows for customization of configuration data based on the current session.
1. Use the configuration policy block to create and initialize a BCOVAMCAnalyticsPolicy instance with `-[initWithHeartbeatConfigurationPolicy:]`. The BCOVAMCAnalyticsPolicy object is used to create the BCOVAMCSessionConsumer.
1. After the playback controller is created, call the add session consumer method, `-[addSessionConsumer:]`, to add the AMC session consumer.

### Marketing Cloud

This example uses media tracking.

```
[1] BCOVAMCMediaSettingPolicy mediaSettingPolicy = ^ADBMediaSettings *(id<BCOVPlaybackSession> session) {

      ADBMediaSettings *settings = [ADBMobile mediaCreateSettingsWithName:<cutomize_setting_name>
[2]                                                                length:0
                                                               playerName:<cutomize_player_name>
                                                                 playerID:<cutomize_player_ID>];
[3]   settings.milestones = @"25,50,75"; // a customization.
      return settings;

    };

[4] BBCOVAMCAnalyticsPolicy *mediaPolicy = [[BCOVAMCAnalyticsPolicy alloc] initWithMediaSettingsPolicy:mediaSettingPolicy];

    BCOVAMCSessionConsumer *sessionConsumer = [BCOVAMCSessionConsumer mediaAnalyticsConsumerWithPolicy:mediaPolicy
                                                                                              delegate:self];

    BCOVPlayerSDKManager *manager = [BCOVPlayerSDKManager sharedManager];
    id<BCOVPlaybackController> controller = [manager createPlaybackController];
    controller.delegate = self;
    [self.view addSubview:controller.view];
   
[5] [controller addSessionConsumer:sessionConsumer];       

    NSString *policyKey = <your-policy-key>;
    NSString *accountId = <your-account-id>;
    NSString *videoID = <your-video-id>;
    BCOVPlaybackService *service = [[BCOVPlaybackService alloc] initWithAccountId:accountId
                                                                        policyKey:policyKey];
    [service findVideoWithVideoID:videoID
                       parameters:nil
                       completion:^(BCOVVideo    *video,
                                    NSDictionary *jsonResponse,
                                    NSError      *error) {

      [controller setVideos:@[ video ]];
      [controller play];

    }];
```

1. Create the media settings policy block which will be called at the start of each playback session. The policy allows for customization of media settings based on the current session.
1. When creating the ADBMediaSettings instance with a class method of ADBMobile `+[mediaCreateSettingsWithName:length:playerName:playerID:]`, you can set **video length** to **0**. The Omniture plugin will update it later.
1. Add optional settings such as milestones. 
1. Use the media settings policy block to create and initialize a BCOVAMCAnalyticsPolicy instance with `-[initWithMediaSettingsPolicy:]`. The policy object is used to create an AMC session consumer for Adobe media tracking as `+[mediaAnalyticsConsumerWithPolicy:delegate:]`.
1. After the playback controller is created, call the add session consumer method, `-[addSessionConsumer:]`, to add the AMC session consumer.

## Adobe-Marketing-Cloud.git

To fetch the Adobe Marketing Cloud libraries, download the following:

```
https://github.com/Adobe-Marketing-Cloud/media-sdks/archive/refs/tags/ios-v2.3.0.zip
https://github.com/Adobe-Marketing-Cloud/mobile-services/archive/refs/tags/v4.21.2-iOS.zip
```

## Support

If you have questions, need help or want to provide feedback, please use the [Support Portal](https://supportportal.brightcove.com/s/login/) or contact your Account Manager.  To receive notification of new SDK software releases, subscribe to the Brightcove Native Player SDKs [Google Group](https://groups.google.com/g/brightcove-native-player-sdks).

[adobeheartbeat]: https://github.com/Adobe-Marketing-Cloud/video-heartbeat/
[adobemobileservice]: https://github.com/Adobe-Marketing-Cloud/mobile-services
[bcovsdk]: https://github.com/brightcove/brightcove-player-sdk-ios
[forum]: https://groups.google.com/forum/#!forum/brightcove-native-player-sdks

---
layout: page_v2
title: Google Ad Manager Integration
description: Integration of Prebid Rendering module whith Google Ad Manager  
sidebarType: 2
---

# AppLovin MAX Integration

The integration of Prebid Mobile with AppLovin MAX assumes that publisher has an AdMob account and has already integrated the AppLovin MAX SDK into the app.

See the [AppLovin MAX Documentation](https://dash.applovin.com/documentation/mediation/ios/getting-started/integration) for the MAX integration details.

## MAX Integration Overview

![Rendering with GAM as the Primary Ad Server](/assets/images/prebid-mobile/modules/rendering/prebid-in-app-bidding-overview-max.png)

**Steps 1-2** Prebid SDK makes a bid request. Prebid server runs an auction and returns the winning bid.

**Step 3** MAX SDK makes an ad request. MAX returns the waterfal with respective placements.

**Step 4** For each prebid's placement, the MAX SDK sequentially instantiates an adapter. 

**Step 5** The adapter verifies the targeting keywords of the winning bid and the custom properties of the given placement. If they match the adapter will render the winning bid. Otherwise, adpater will fail with "no ad" immediately and the next placement will instantiate the same adapter but for another set of server parpams. 
  
Prebid Mobile supports these ad formats:

- Display Banner
- Display Interstitial
- Video Interstitial 
- Rewarded Video
- Native

They can be integrated using these API categories:

- [**Banner API**](#banner-api) - for *Display* Banner
- [**Interstitial API**](#interstitial-api) - for *Display* and *Video* Interstitials
- [**Rewarded API**](#rewarded-api) - for *Rewarded Video*
- [**Native API**](#native-ads) - for *Native Ads*


## Banner API

Integration example:

``` swift
// 1. Create MAAdView
adBannerView = MAAdView(adUnitIdentifier: maxAdUnitId)
adBannerView?.delegate = self
        
// 2. Create MAXMediationBannerUtils
mediationDelegate = MAXMediationBannerUtils(adView: adBannerView!)
    
// 3. Create MediationBannerAdUnit
adUnit = MediationBannerAdUnit(configID: prebidConfigId,
                                       size: adUnitSize,
                                       mediationDelegate: mediationDelegate!)
    
// 4. Make a bid request
adUnit?.fetchDemand { [weak self] result in

    // 6. Make an ad request to MAX
    self?.adBannerView.loadAd()
}
```

#### Step 1: Create MAAdView

This step is totally the same as for pure [MAX integration](https://dash.applovin.com/documentation/mediation/ios/getting-started/banners). You don't have to make any modifications here.


#### Step 2: Create MAXMediationBannerUtils

The `MAXMediationBannerUtils` is a helper class, wich performs certain utilty work for the `MediationBannerAdUnit`, like passing the targeting keywords to the adapters and checking the visibility of the ad view.

#### Step 3: Create MediationBannerAdUnit

The `MediationBannerAdUnit` is part of Prebid mediation API. This class is responsible for making bid request and providing the winning bid and targeting keywords to mediating SDKs.  

#### Step 4: Make bid request

The `fetchDemand` method makes a bid request to prebid server and provides a result in a completion handler.

#### Step 5: Make an Ad Reuest

Now you should make a regular MAX's ad request. Everything else will be handled by prebid adapters.

## Interstitial API

Integration example:

``` swift
// 1. Create MAInterstitialAd
interstitial = MAInterstitialAd(adUnitIdentifier: maxAdUnitId)
interstitial.delegate = self

// 2. Create MAXMediationInterstitialUtils
mediationDelegate = MAXMediationInterstitialUtils(interstitialAd: interstitial!)

// 3. Create MediationInterstitialAdUnit
adUnit = MediationInterstitialAdUnit(configId: prebidConfigId,
                                             minSizePercentage: CGSize(width: 30, height: 30),
                                             mediationDelegate: mediationDelegate!)

// 4. Make a bid request
adUnit?.fetchDemand { [weak self] result in
    guard let self = self else { return }
            
    guard result == .prebidDemandFetchSuccess else {
        self.fetchDemandFailedButton.isEnabled = true
        return
    }
    
    // 6. Make an ad request to MAX
    self.interstitial?.load()
})
```

In order to make a `multiformat bid request`, set the respective values into the `adFormats` property.

``` swift
// Make bid request for video ad                                     
adUnit?.adFormats = [.video]

// Make bid request for both video amd disply ads                                     
adUnit?.adFormats = [.video, .display]

// Make bid request for disply ad (default behaviour)                                     
adUnit?.adFormats = [.display]

```

#### Step 1: Create MAInterstitialAd 

This step is totally the same as for pure [MAX integration](https://dash.applovin.com/documentation/mediation/ios/getting-started/interstitials). You don't have to make any modifications here.


#### Step 2: Create MAXMediationBannerUtils

The `MAXMediationBannerUtils` is a helper class, wich performs certain utilty work for the `MediationBannerAdUnit`, like passing the targeting keywords to the adapters and checking the visibility of the ad view.

#### Step 3: Create MediationInterstitialAdUnit

The `MediationInterstitialAdUnit` is part of the prebid mediation API. This class is responsible for making a bid request and providing a winning bid to the mediating SDKs.  

#### Step 4: Make bid request

The `fetchDemand` method makes a bid request to prebid server and provides a result in a completion handler.

#### Step 5: Make an Ad Reuest

Now you should make a regular MAX's ad request. Everything else will be handled by GMA SDK and prebid adapters.

#### Steps 7: Display an ad

Once you receive the ad it will be ready for display. Folow the [MAX instructions](https://dash.applovin.com/documentation/mediation/ios/getting-started/interstitials#showing-an-interstitial-ad) about how to do it. 

## Rewarded API

Integration example:

``` swift
// 1. Get an instance of MARewardedAd
rewarded = MARewardedAd.shared(withAdUnitIdentifier: maxAdUnitId)
rewarded.delegate = self

// 2. Create MAXMediationRewardedUtils
mediationDelegate = MAXMediationRewardedUtils(rewardedAd: rewarded!)
    
// 3. Create MediationRewardedAdUnit
adUnit = MediationRewardedAdUnit(configId: prebidConfigId, mediationDelegate: mediationDelegate!)
        
// 4. Make a bid request
adUnit?.fetchDemand { [weak self] result in
    guard let self = self else { return }
    
    // 5. Make an ad request to MAX
    self.rewarded?.load()
}
```

The way of displaying the rewarded ad is totally the same as for the Interstitial Ad. 

To be notified when user earns a reward follow the [MAX intructions](https://dash.applovin.com/documentation/mediation/ios/getting-started/rewarded-ads#loading-a-rewarded-ad).

#### Step 1: Get an instance of MARewardedAd

This step is totally the same as for pure [MAX integration](https://dash.applovin.com/documentation/mediation/ios/getting-started/rewarded-ads). You don't have to make any modifications here.


#### Step 2: Create MAXMediationRewardedUtils

The `MAXMediationRewardedUtils` is a helper class, wich performs certain utilty work for the `MediationRewardedAdUnit`, like passing the targeting keywords to the adapters.

#### Step 3: Create MediationInterstitialAdUnit

The `MediationRewardedAdUnit` is part of the prebid mediation API. This class is responsible for making a bid request and providing a winning bid and targeting keywords to the adapters.  

#### Step 4: Make bid request

The `fetchDemand` method makes a bid request to the prebid server and provides a result in a completion handler.

#### Step 5: Make an Ad Reuest

Now you should make a regular MAX's ad request. Everything else will be handled by GMA SDK and prebid adapters.

#### Steps 6: Display an ad

Once the rewarded ad is recieved you can display it. Folow the [MAX instructions](https://dash.applovin.com/documentation/mediation/ios/getting-started/rewarded-ads#showing-a-rewarded-ad) for the details. 

## Native Ads

Integration example:

``` swift
// 1. Create MANativeAdLoader
nativeAdLoader = MANativeAdLoader(adUnitIdentifier: maxAdUnitId)
nativeAdLoader?.nativeAdDelegate = self

// 2. Create MAXMediationNativeUtils
mediationDelegate = MAXMediationNativeUtils(nativeAdLoader: nativeAdLoader!)

// 3. Create and configure MediationNativeAdUnit
nativeAdUnit = MediationNativeAdUnit(configId: prebidConfigId,
                                     mediationDelegate: mediationDelegate!)

nativeAdUnit.setContextType(ContextType.Social)
nativeAdUnit.setPlacementType(PlacementType.FeedContent)
nativeAdUnit.setContextSubType(ContextSubType.Social)
 
// 4. Set up assets for bid request
nativeAdUnit.addNativeAssets(nativeAssets)

// 5. Set up event tracker for bid request
nativeAdUnit.addEventTracker(eventTrackers)

// 6. Make a bid request
nativeAdUnit.fetchDemand { [weak self] result in

    // 7. Make an ad request to MAX
    self?.nativeAdLoader?.loadAd(into: self?.createNativeAdView())
}
```

#### Step 1: Create MANativeAdLoader

Prepare the `MANativeAdLoader` object before you make a bid request. It will be needed for prebid mediation utils. 

#### Step 2: Create MAXMediationNativeUtils

The `MAXMediationNativeUtils` is a helper class, wich performs certain utilty work for `MediationNativeAdUnit`, like passing the targeting keywords to adapters and checking the visibility of the ad view.

#### Step 3: Create and configure MediationNativeAdUnit

The `MediationNativeAdUnit` is part of the prebid mediation API. This class is responsible for making a bid request and providing a winning bid and targeting keywords to the adapters. Fot the better targetting you should provide additional properties like `conteaxtType` and `placemantType`. 
 
#### Step 4: Set up assets for bid request

The bid request for native ads should have the description of expected assets. The full spec for the native template you can find in the [Native Ad Specification from IAB](https://www.iab.com/wp-content/uploads/2018/03/OpenRTB-Native-Ads-Specification-Final-1.2.pdf). 

The example of creating the assets array:

```
let image = NativeAssetImage(minimumWidth: 200, minimumHeight: 50, required: true)
image.type = ImageAsset.Main

let icon = NativeAssetImage(minimumWidth: 20, minimumHeight: 20, required: true)
icon.type = ImageAsset.Icon

let title = NativeAssetTitle(length: 90, required: true)

let body = NativeAssetData(type: DataAsset.description, required: true)

let cta = NativeAssetData(type: DataAsset.ctatext, required: true)

let sponsored = NativeAssetData(type: DataAsset.sponsored, required: true)

return [icon, title, image, body, cta, sponsored]
```

#### Step 5: Set up event tracker for bid request

The bid request for mative ads may have a descrition of expected event trackers. The full spec for the Native template you can find in the [Native Ad Specification from IAB](https://www.iab.com/wp-content/uploads/2018/03/OpenRTB-Native-Ads-Specification-Final-1.2.pdf). 

The example of creating the event trackers array:

```
let eventTrackers = [
    NativeEventTracker(event: EventType.Impression,
                       methods: [EventTracking.Image,EventTracking.js])
]
```

#### Step 6: Make a bid request

The `fetchDemand` method makes a bid request to prebid server and provides a result in a completion handler.
    
#### Step 7: Load AdMob Native ad
    
Now just load a native ad from MAX according to the [MAX instructions](https://dash.applovin.com/documentation/mediation/ios/getting-started/native-manual#load-the-native-ad). 
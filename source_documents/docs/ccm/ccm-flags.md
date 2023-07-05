---
title: Glass iOS CCM Flags
navTitle: CCM Flags
---

# Glass iOS CCM Flags

This document contains all the CCM Flag keys used in the app

CCM Flag key shall follow dot notation pattern which is in form of alphanumeric characters separated by dots.
It doesn't allow special characters.
E.g. feature.enabled (O), isFeatureEnabled (X)

**NOTE: For clarity, keep this list alphabetized**

| Flag Key | Description | Value format |
|---|---|---|
| `account.address.bypassValidationEnabled` | Determines whether the user can bypass address validation. | Boolean |
| `account.address.maxNumberAllowed` | Determines the max number of addresses a user can have. Defaults to 20. | Int |
| `account.EAE.accountDeletion.isEnabled` | Determines whether the user can have the option to deactivate account. | Boolean |
| `account.profileBuilder.vehicles.isEnabled` | Determines whether the user can access thier vehicles profile
| `cart.affirm.enabled` | Controls the visibility of the Affirm option on the cart page. | Boolean |
| `cart.capOneApplyBanner.enabled` | Controls the visibility of the Capital One credit card application banner on the cart page. | Boolean |
| `cart.capOneRewardBanner.enabled` | Controls the visibility of the rewards points banner on the cart page for users with a linked Capital One credit card. | Boolean |
| `cart.carePlan.enabled` | Controls the visibility of the carePlan association on the cart page, checkout page and thank you page. Defaults to false. | Boolean |
| `cart.lastCall.useProductGridResponseV1.enabled` | Controls querying for `productGridResponseV1` in LastCall content Tempo query. Defaults to false. | Boolean |
| `cart.reservationExpiryMessage.windowDurationInSeconds` | Determines for how long "reservation expired" messaging should be displayed after a user's reservation has expired. | Int |
| `cart.saveForLater.enabled` | Determines whether the save for later (SFL) feature is enabled. Controls the visibility of saved for later items and the "Save for later" action on cart items. | Boolean |
| `cart.saveForLater.initialItemsPageSize` | Determines how many items to request and display in the initial state of the SFL items module on the cart page. | Int |
| `cart.saveForLater.subsequentItemsPageSize` | Determines how many items to request in each pagination call after the user taps "View all items" in the SFL items module on the cart page. | Int |
| `categoryPages.dropCategoryName.enabled` | Determines whether to drop category name from URL before setting the pageId. | Boolean |
| `checkin.isAlcoholPickupEnabled` | Indicates if Alcohol Pickup aka "Liquor Box" is enabled. | Boolean |
| `checkin.isAnnualEventEnabled` | Indicates if Annual Event aka "AE" is enabled. The AE curbside pickup experience has dark blue background with falling glitter animation. | Boolean |
| `checkin.isBarcodeBrightnessEnabled` | Indicates if the pickup screens featuring a barcode in the Curbside Returns CheckIn feature will automatically increase the screen brightness. | Boolean |
| `checkin.isHideFeedbackButtonEnabled` | Indicates if the Feedback button will be shown at the end of a pickup. | Boolean |
| `checkin.isHubOrSpokeEnabled` | Indicates if Hub & Spoke pickup is enabled. | Boolean |
| `checkin.isIAMCompletionCheckinEnabled` | Indicates if the `InAppMessagingAPI` will be used to present the final screen of the pickup flow. | Boolean |
| `checkin.isIAMParkingFlowEnabled` | Indicates if the `InAppMessagingAPI` will be used to present the parking details screen of the pickup flow. | Boolean |
| `checkin.isIAMSelectionCheckinEnabled` | Indicates if the `InAppMessagingAPI` will be used to present the CheckIn selection screen of the pickup flow. | Boolean |
| `checkin.isIntlResponseChangeEnabled` | Indicates if the Pegasus response should be handled differently for international (CA & MX) markets. | Boolean |
| `checkin.isNextPickupFlowEnabled` | Indicates if the "next pickup" flow is enabled. If enabled, the customer will be prompted to check-in for their next order after completing a pickup at a store where they have more than one pickup to complete. | Boolean |
| `checkin.isReturnCheckInEnabled` | Indicates if Curbside Returns CheckIn feature is enabled. | Boolean |
| `checkin.isUnmarkedParkingSpotEnabled` | Indicates if the unmarked aka "not listed" parking spot is shown in the curbside pickup experience. | Boolean |
| `checkout.customerChoiceSubs.enabled` | Indicates if customer choice substitution experience is accessible from the `Checkout` screen or not. | Boolean |
| `checkout.tipping.enabled` | Controls the visibility of tipping section in review order screen. Defaults to false. | Boolean |
| `checkout.donation.enabled` | Controls the visibility of donation section in review order screen. Defaults to false. | Boolean |
| `checkout.walmartPlus.splashPhase2.enabled` | Enables and disables the check for CC/DC in order to display the Wplus Membership Banner. Defaults to false. | Boolean |
| `feature.appsflyer.resolveShortLinks.enabled` | Enables and disables the redirection of OneLink short links. Defaults to `true` | Boolean |
| `feature.cancellation.instantV2.enabled` | Indicates if Cancellation V2 is enabled. Defaults to `false`. | Boolean |
| `feature.checkin.isArriveButtonEnabled` | Indicates if the "I've arrived" button is shown in the in-store pickup experience. | Boolean |
| `feature.checkin.isBarcodeImageEnabled` | Indicates if the barcode image will be shown in the in-store pickup experience. | Boolean |
| `feature.checkin.isLGPickupEnabled` | Indicates if Lawn & Garden pickup is enabled. | Boolean |
| `feature.checkin.isManualArrivalEnabled` | Determines if the `arrived` service needs to be called when the `I have arrived` button is pressed in `In Store` and in the `I have parked` screen in Curbside flow. Defaults to false. | Boolean |
| `feature.checkin.isMembershipBottomSheetEnabled` | Indicates if the Walmart+ acquisition bottom sheet is enabled. | Boolean | 
| `feature.checkin.isPegasusApiProxyEnabled` | Indicates if the API proxy URL or the legacy URL should be used to reach Pegasus. | Boolean |
| `feature.checkin.isPickUpInstructionsEnabled` | Determines whether to show a brief how to pickup description in pickup selector. Defaults to false. | Boolean |
| `feature.checkin.isSummarizedInstructionsEnabled` | Indicates if Summarized Instructions on in-store pickup screen is enabled. | Boolean |
| `feature.checkin.isWplusMembershipTempoBannerEnabled` | Indicates if the Walmart+ acquisition banner is shown on the parking confirmation screen. | Boolean |
| `feature.purchaseHistory.checkin.enabled` | Determines whether all Checkin action button can be displayed on the `PurchaseHistory` order detail screen. Defaults to `true`. | Boolean |
| `feature.purchaseHistory.itemsReviewModule.enabled` | Determines if "items review" module is enabled. Defaults to `false`. | Boolean |
| `feature.purchaseHistory.liveChat.enabled` | Determines whether an action button to open `Converse` live chat can be displayed on the `OrderDetails` screen. Defaults to `true`. | Boolean |
| `feature.purchaseHistory.liveShopping.enabled` | Determines whether an action button to open `SparkShopper` can be displayed on the `PurchaseHistory` and `OrderDetails` screens. Defaults to `true`. | Boolean |
| `feature.purchaseHistory.lostAfterDelivery.url` | Delivery help URL used in order status tracker `PurchaseHistory` and `OrderDetails` screens. Defaults to `""`. | String |
| `feature.purchaseHistory.projectHoT.enabled` | Indicates if project Here-or-There (HoT) changes are applied in Purchase History. Defaults to `false`. | Boolean |
| `feature.purchaseHistory.receiptScanning.enabled` | Determines whether receipt scanning is enabled. Defaults to `true`. | Boolean |
| `feature.purchaseHistory.reorderAll.enabled` | Determines whether an order status tracker "Reorder all" action button can be displayed on the `PurchaseHistory` screen. Defaults to `false`. | Boolean |
| `feature.purchaseHistory.reorderEssentials.enabled` | Determines whether a "Reorder your essentials" action button can be displayed on the `PurchaseHistory` screen. Defaults to `true`. | Boolean |
| `feature.purchaseHistory.showSearchFilters.enabled` | Determines whether the pre-defined search filters can be displayed on the `PurchaseHistory` screen. Defaults to `false`. | Boolean |
| `feature.purchaseHistory.showStartReturn.enabled` | Determines whether a "Start a return" action button can be displayed on the `PurchaseHistory` screen. Defaults to `true`. | Boolean |
| `feature.scanner.global.enabled` | Determines whether global scanner is enabled. Defaults to `false`. | Boolean |
| `feature.scanner.global.outOfStoreMessaging.enabled` | Determines whether the out of store onboarding bottom sheet is displayed. Defaults to `true`. | Boolean |
| `feature.scanner.global.inStoreMessaging.enabled` | Determines whether the in-store onboarding bottom sheet is displayed. Defaults to `true`. | Boolean |
| `feature.scanner.global.healthAndWellnessHandler.enabled` | Determines whether the health and wellness (pharmacy) handler is enabled. Defaults to `false`. | Boolean |
| `feature.scanner.barcode.tracking.enabled` | Determines whether regular barcode tracking is enabled. Defaults to `true`. | Boolean |
| `feature.scanner.qrcode.tracking.enabled` | Determines whether QR code tracking is enabled. Defaults to `false`. | Boolean |
| `feature.storeMaps.enabled` | Controls whether the StoreMaps can be initiated from the entry points or not. Defaults to false. | Boolean |
| `feature.tippingAndFeedback.tippingPaymentMethod.enabled` | Determines if tipping payment method should be used instead of standard credit card payment method in tipping section of Order Details Page. Defaults to `false`. | Boolean |
| `feature.projectSky.badges.enabled` | Controls the visibility of benefit provider badges in product tile. Defaults to `false`. | Boolean |
| `find.decide.legacySpinner.enabled` | Determines whether Search & Item page should display the legacy loading spinner type. Defaults to false. | Boolean |
| `fulfillmentETE.customerChoice.checkoutEntryEnabled` | Determines whether the entry point to the new Customer Choice Substitution experience is shown on the `Checkout` screen. Defaults to `false`. | Boolean |
| `fulfillmentETE.customerChoice.orderDetailsEntryEnabled` | Determines whether the entry point to the new Customer Choice Substitution experience is shown on the `OrderDetails` screen. Defaults to `false`. | Boolean |
| `identity.authenticatedDelete.passwordFlow.enabled` | Intended only for international markets (Canada & Mexico) to use password authentication to confirm identity when deactivating an account. Defaults to `false` | Boolean |
| `identity.fyp.enableEmailLink` | Flag intended only for international markets (Canada & Mexico) that does not support OTP for resetting password. Requests e-mail with link to reset password to be sent and shows screen to inform user e-mail has been sent when successful. Defaults to `false` | Boolean |
| `item.carePlans.enabled` | Controls the visibility of CarePlans (Protection Plans & Home Service) section on Item Details Screen. Defaults to false. | Boolean |
| `item.completeTheLook.enabled` | Determines whether the "Complete The Look" module is enabled in Item Page. Defaults to `false` | Boolean |
| `item.completeTheLook.layoutVersion` | Determines the layout version to use for the "Complete the Look" section. Defaults to `1` | Int |
| `item.petrx.enabled` | When enabled, it shows PetRx features on item details page. Defaults to false. | Boolean |
| `item.egiftcard.endpoint` | Controls the end-point/URL of the e-gift card external web page. | String |
| `item.fitPredictor.enabled` | Controls whether Fit Predictor © section is shown in the Item Page ragardless of whether the item is fit predictable or not. Defaults to `true`. | Boolean |
| `item.ffClarityStoreSelector.enabled` | Controls the visibility of store selector link and interactibility with that link for Fulfillment Clarity. Defaults to `false` | Boolean |
| `item.ffClarityAddressSelector.enabled` | Controls the visibility of address selector link and interactibility with that link for Fulfillment Clarity. Defaults to `false` | Boolean |
| `item.mpStoreSelector.enabled` | Controls the visibility of store selector link and interactibility with that link for MPPickup Store. Defaults to false. | Boolean |
| `item.fulfillmentControl.enabled` | Determines whether the "Item Level Fulfillment Control" module is enabled on item page. Defaults to false. | Boolean |
| `item.holidayQueueBanner.enabled` | Controls whether the Holiday Queue Timer banner is shown in the Item Details Page. Defaults to `true` | Boolean |
| `item.holidayReturns.enabled` | Determines whether to show holiday return policy updates on item page. Defaults to false. | Boolean |
| `item.imageZoom.enabled` | Determines whether to enable zoom on the ItemPage image.Defaults to true. | Boolean |
| `item.imageZoom.numberOfLargeImagesToPreload` | Controls the number of large images to be pre-loaded on Item page. Default to 4. | Int |
| `item.locationExpansion.enabled` | Determines whether to enable location expansion of fulfillment on ItemPage. Defaults to false. | Boolean |
| `item.nonConfigBundles.enabled` | Determines whether to enable NonConfig bundle on ItemPage. Defaults to false. | Boolean |
| `item.PAC.enabled` | Determines if PAC (Post Add-to-Cart) is enabled; shows suggestions after user adds item to cart. Default false. | Boolean |
| `item.relatedSearch.enabled` | Determines whether to show Related searches modules in itempage. Defaults to false. | Boolean |
| `item.sizeGuideTab.enabled` | Controls whether the user is presented with a tabbed interface rather than full screen when taping on the CTA of the Fit Predictor © widget. Defaults to `false`. | Boolean |
| `item.softBundles.enabled` | Determines whether to enable Soft Bundles module on ItemPage. Defaults to false. | Boolean |
| `item.subscription.enabled` | Determines whether to show purchase options radio buttons for item subscription or not. Defaults to false. | Boolean |
| `item.subscriptions.variation` | Determines whether nonWplusMembers can subscribe. 0 value requires membership, 1 value does not. Defaults to 0. | Int |
| `item.subscriptionsOptimizations.enabled` | Determines whether to show the optimized subscription experience, with green pricing and subscription how it works panel. Defaults to false. | Boolean |
| `item.variants.subscription.enabled` | Determines whether product variants are enabled for subscription or not. Defaults to false. | Boolean |
| `item.viewSimilarCTA.enabled` | Determines whether to show view similar CTA below item image. Defaults to false. | Boolean |
| `item.virtualPack.enabled` | Determines whether to show virtual pack info. Defaults to false. | Boolean |
| `item.urlForCache.enabled` | Determines whether to enable Torbit caching on the ItemPage product details request. Defaults to true. | Boolean |
| `item.buyBoxSuppression.enabled` | Determines whether to hide buybox related info on item page. Defaults to false. | Boolean |
| `item.walmartCoFunding.enabled` | Determines whether to show walmart coFunding info on item page. Defaults to false. | Boolean |
| `item.warningMessage.enabled` | Determines whether to show a warning message on ItemPage when user comes through deeplink. Defaults to false. | Boolean |
| `item.wirelessPrepaid.enabled` | Controls the visibility of prepaid wireless module on Item page. Defaults to false. | Boolean |
| `item.writeAReview.enabled` | Controls the visibility of write a review module on Item page. Defaults to false. | Boolean |
| `item.reviewMore.enabled` | Controls the visibility of Review More Items module on Write a Review page. Defaults to false. | Boolean |
| `item.wyswyg.fulfillmentPrefEnabled` | Controls the fulfillment preference flag (Store or Shipping) in SEM during ATC. Defaults to true. | Boolean |
| `onboarding.demoModeEnabled` | Determines whether to show demostores to user. Defaults to false. | Boolean |
| `marketplace.cart.isAddressButtonRelocatedEnabled` | Indicates if Add/Edit Address Button in Cart is moved to the title in a hyperlink to make space for marketplace shipping options button. | Boolean |
| `marketplace.isContactSellerEnabled` | Indicates if views using Marketplace contact seller methods are enabled. | Boolean |
| `marketplace.isEnabled` | Indicates if ItemDetails shows certain Marketplace details and if a special header is added to Marketplace-related network requests. | Boolean |
| `marketplace.isWriteAReviewEnabled` | Indicates if write a review is enabled for Marketplace sellers and products in Item Details and Order Details. | Boolean |
| `onboarding.idfa.messagingImageName` | IDFA message placeholder image name. Default "OnboardingIDFA"(other option is "OnboardingIDFANew") | String |
| `onboarding.idfaMessagingEnabled` | Determines whether to show idfa prompt to user to request tracking. Defaults to false. | Boolean |
| `onboarding.stateRestorationEnabled` | Determines whether to retain onboarding state when incomplete. Defaults to false. | Boolean |
| `onboarding.optOut.dsCardExclusion` | Determines whether to show OptOut for Directed Spend card users. Defaults to false. | Boolean |
| `onboarding.optOut.ebtExclusion` | Determines whether to show OptOut for EBT card users. Defaults to false. | Boolean |
| `onboarding.optOut.inHomeExclusion` | Determines whether to show OptOut for InHome users. Defaults to false. | Boolean |
| `orderDetails.buyNow.enabled` | Determines whether a "Buy now" action button can be displayed in the item level on the `OrderDetails` screen. Defaults to `false`. | Boolean |
| `orderDetails.nativeReshop.enabled` | Determines whether the entry point to the Reshop experience is shown on the `OrderDetails` screen. Defaults to `false`. | Boolean |
| `orderDetails.orderSubstitutions.enabled` | Determines if substitution options can be displayed on the `OrderDetails` screen. Defaults to `false`. | Boolean |
| `orderDetails.refundsAndReplacementsCTA.enabled` | Controls text displayed on an action button on the `OrderDetails` screen. The text will be "Start a return" if false, or "Refunds and replacements" if true. Defaults to `false`. | Boolean |
| `orderDetails.showStartReturnOnTop.enabled` | Determines placement of "Start a return" button on the `OrderDetails` screen. Placement will be at top if `true`. Placement will be near bottom if `false`. Defaults to `false`. | Boolean |
| `orderDetails.substitutionsToReshopFlow.enabled` | Determines whether the transition from Substitutions to Reshop experience happens from the `OrderDetails` screen. Defaults to `false`. | Boolean |
| `orderDetails.wPlusRewardsBanner.enabled` | In CheckIn plugin, indicates if the customer should see the Cashback experience on the Walmart+ acquisition banner. In PurchaseHistory plugin, indicates if Walmart+ rewards banners are shown on Order Details Page. Defaults to `false`. | Boolean |
| `paymentTransaction.klarna.enabled` | Enable Buy Now Pay Later (BNPL) payment method, Klarna.  Added for Canada.  Defaults to `false`. | Boolean |
| `platform.app.nudgeUpdate.model.waitDurations` | Upgrade nudge model redisplay time duration, ex: "0,24,168" in hours. | String |
| `platform.app.nudgeUpdate.enabled` | Determines whether to show Upgrade nudge to user. Defaults to false. | Boolean |
| `platform.app.nudgeUpdate.reminderModel.waitDurations` | Upgrade nudge reminder model redisplay time duration, ex: "336" in hours. | String |
| `platform.app.forceUpgrade.enabled` | Determines whether to show force upgrade screen to user. Defaults to false. | Boolean |
| `platform.attribution.conversionValueUpdatePeriod` | Period in days to send conversion value to Apple's SKADNetwork API | Int |
| `platform.fraud.bstc.enabled` | Determines whether to use the BSTC cookie as the session ID for ThreatMetrix profiling. Defaults to false. | Boolean |
| `platform.fraud.orgId` | Org ID used by the ThreatMetrix SDK to uniquely identify the app. Defaults to hardcoded value. | String |
| `platform.fraud.session.ttl` | Time in minutes during which the ThreatMetrix session ID is considered valid. Defaults to 1440. | Int |
| `platform.fraud.success.logging.enabled` | Determines whether additional debugging logs, such as successful operations, should be sent during ThreatMetrix profiling. Defaults to false. | Boolean |
| `platform.geofence.accessPoint.defaultRadiusMeters` | Capatilities which are promoted fuel stations) have this radius by default | Double |
| `platform.geofence.minimumAccuracyForMonitoring` | User locations with accuracy less that this will be discarded | Double |
| `platform.image.bound.download.enabled` | Images will be downloaded using odnBound with correct aspect ratio and optimise image size | Boolean |
| `platform.poi.enabled`| Geofenc API will not `start` unless this is on. | Bool |
| `platform.poi.nearBy.poiCacheTtlDays` | Lifetime in days of the poi cache | Int |
| `platform.poi.nearBy.poiScoreTtlMilliseconds` | The TTL in milliseconds for a score for any geofence result. Default to 1500000 ms (25 minutes) | Int64 |
| `platform.poi.nearBy.poiRefreshDisplacementThresholdKm` | POI cache will add a new entry to store the users nearby pois in after they have moved this distance | Int |
| `platform.poi.nearBy.poiRadiusMiles` | Default request radius when retrieving pois from OL | Int |
| `platform.poi.storeProperty.enabled` | When enabled each store will also have a storeproperty geofence created for it | Bool |
| `platform.poi.storeProperty.offsetMeters` | The size to use for the store property (beyond the POI's given radius) | Double |
| `platform.poi.tracking.filter` | The allowed store detail types ["1","2","4","10","45"]| `Array<String>` |
| `platform.poi.tracking.accessPointTypeFilter` | Allowed access point types that can be promoted to top level poi | `Array<String>` |
| `platform.poi.radius.minThreshold` | Used to clamp the radius that we get back from the OL | Double |
| `platform.poi.radius.maxThreshold` | Used to clamp the radius that we get back from OL | Double |
| `platform.waitingRoom.refreshInterval.min` | The lower bound of the time required before retrying a `429` (rate limiting) error. Defaults to 15. | Int |
| `platform.waitingRoom.refreshInterval.max` | The upper bound of the time required before retrying a `429` (rate limiting) error. Defaults to 30. | Int |
| `protectionPlans.feature.isEnabled` | Determines if native `ProtectionPlans` views for protection plan hub and details are enabled, otherwise a web view is used. Defaults to `false`. | Bool |
| `purchaseHistory.accReschedule.enabled` | Determines whether an ACC reschedule action button can be displayed on the order status tracker on `Account`, `PurchaseHistory`, and `OrderDetails` screens. Defaults to `false`. | Boolean |
| `purchaseHistory.accScheduling.enabled` | Indicates if ACC feature is available in `PurchaseHistory` and `OrderDetails`. Defaults to `false`. | Boolean |
| `substitutions.chargeForSubs.enabled` | Indicates if additional copy about charging full amount for substituted items needs to be displayed or not. Default value is `true`. | Boolean |
| `substitutions.optInPriceClarity.enabled` | Indicates if additional copy about Substitution Details will be displayed on Review Order Page. Default value is `false`. | Boolean |
| `substitutions.priceQuantityClarity.enabled` | Indicates if price clarity feature is enabled on Substitutions flows (CXO, Amends). Default value is `false`. | Boolean |
| `feedback.inAppRatingEnabled` | Determines whether to show app raing prompt. Defaults to false. | Boolean |
| `feedback.scanAndGo.inAppRatingEnabled` | Determines whether to show app raing prompt in scan and go feature specific. Defaults to false. | Boolean |
| `feedback.appStoreUrl` | Determines app store url in case of navigating to app store. | String |
| `platform.dealsshortcut.enabled`| Determines if the deals is enabled for app shortcut.  | Boolean |
| `tooltip.store.nav`| Whether to show callout for store selector on Home.  Defaults false except for Mexico.  | Boolean |
| `item.acc.tireScheduling.enabled` | Controls the visibility of ACC Tire Scheduling. Defaults to false. | Boolean |
| `item.brandLinks.enabled` | When enabled, the brand name in Item page becomes a link, and when tapped, it takes us to search page with search results for products from the brand. Defaults to false. | Boolean |
| `item.richMedia.videosEnabled` | When enabled, it shows videos in the Hero Image section of item page. Defaults to false. | Boolean |
| `feature.itemPage.digitalRewards.enabled` | When enabled, it shows digital rewards in the iBotta Digital Reward section of item page. Defaults to false. | Boolean |
| `item.personalisedItems.enabled` | When enabled, it shows personalised items with personlised button on CTA, and takes the user on the browser on tap. Defaults to false. | Boolean |
| `item.refurbished.enabled` | When enabled, it shows refurbished section on item details page. Defaults to false. | Boolean |
| `item.smile.variation` | There are three possible cases handled **0 =  control**, **1 = variation 1** item page and carousel p13n show the price and the discount price with the `Was` word with no strikethrough , e.g. $xx.xx Was xx.xx. **2 = variation 2** item page and carousel p13n show the price with a prefix, BE determine it, usually it says `Now` and the discount price with a strikethrough , e.g. Now $xx.xx ˜˜xx.xx˜˜. **3 = variation 3** item page and carousel p13n show a badge with the legend SAVE + amount, e.g. SAVE $xx.xx. All three variations show the price in green | Int |
| `item.protectionPlan.enabled` | When enabled, it shows protection plan section on item details page. Defaults to false. | Boolean |
| `item.protectionPlan.aboveFulfillment.enabled` | When enabled, it shows protection plan section above ILC module on item details page. Defaults to false. | Boolean |
| `item.eventPricing.enabled` | Determines whether to show special price or price flip data. Defaults to false. | Boolean |
| `item.storeOnlyItem.enabled` | When enabled, it shows store only item section on item details page. Defaults to false. | Boolean |
| `checkin.isArriveButtonEnabled` | Determines whether to show a button to manually notify the arrival to the store for `InStorePickup`. Defaults to false. | Boolean |
| `checkin.isSummarizedInstructionsEnabled` | Determines whether to show a summarized instructions in `InstorePickup`. Defaults to false. | Boolean |
| `checkin.isBarcodeImageEnabled` | Determines whether to show barcode image for every order in `InStorePickup`. Defaults to true. | Boolean |
| `checkin.isUnmarkedParkingSpotEnabled` | Determines whether to show an extra option called `NotListedParkingSpot` in `CurbsidePickup`. Defaults to true. | Boolean |
| `item.outBoxPinchToZoom.enabled` | Enable Outbox Pinch To Zoom feature on itemPage, specifically in `HeroImage` Section. Defaults to false. | Boolean |
| `item.inlinePinchToZoom.enabled` | Enable Inline Pinch To Zoom feature on itemPage, specifically in the `HeroImage` Section. Overrides OutBox Pinch To Zoom if both flags are set to TRUE. Defaults to false. | Boolean |
| `item.inappropriate.review.report.enabled` | Enable flag feature on review section of itemPage, specifically in the `ReviewFeedbackView` Section. Overrides flag display if set to TRUE. Defaults to false. | Boolean |
| `feature.orderDetails.acc.pfs.enabled` | Determines if auto-care-center (ACC) pay-for-service (PFS) changes are enabled. Defaults to `false`. | Boolean |
| `feature.orderDetails.invoicing.enabled` | Determines if invoice details needs to be listed. Defaults to `false`. | Boolean |
| `feature.orderDetails.cancellationEmbeddedWebBrowser.enabled` | It enables an embedded web browser to show the order detail page, so the user can start the cancellation flow from there | Boolean |
| `feature.orderDetails.returnsShortTerm.enabled` | Determines whether to show a button to return a group of products from the same seller | Boolean |
| `feature.orderDetails.nilPickClarity.enabled` | Determines if nil-pick (unavailable items) enhancements are enabled. Defaults to `true`. As the feature is controlled by CPH backend this CCM is just a safety measure to guard the change in the FE. | Boolean |
| `feature.orderDetails.subSellerDisclosure.enabled` | Determines whether to sub seller disclosure enable on ODP. Defaults to `false`. | Boolean |
| `feature.orderDetails.receiptLinking.enabled` | Indicates if the store e-receipts feature is enabled. This feature will add a store receipt to the customer's purchase history when they open the app via a deep link. The customer will also see an Exit Pass on ODP for some period of time. | Boolean |
| `platform.networking.persistedquery.optout.apipath.list` | List of API paths to opt out of persisted query feature; e.q. ["home"]. Defaults to []. | `Array<String>` |
| `platform.background.refresh.scheduler.enabled` | Determines whether to enable a background refresh for a task when app goes in background. Defaults to `false`. | Boolean |
| `platform.background.refresh.scheduler.time` | The earliest time at which to run the task in background refresh. Defaults to 180(3 mins). | Double |
| `item.acc.phase2.enabled` | Determines whether to enable ACC phase 2 or not on item page or PDP. Defaults to `false`. | Boolean |
| `feature.purchaseHistory.convertToPickUp.enabled` | Determines delayed order CTAs should be shown or not in Purchase history and Pickup instead button should be shown or not in order detail page. Defaults to `false`. | Boolean |
| `item.showShopSimilarOnFulfillment.enabled` | Determines whether to enable shopSimilar button in BuyBox fulfillment in item page. Defaults to `false`. | Boolean |
| `platform.networking.remoteImageAccept.enabled` | Determines whether the custom image header accept should be enabled. Defaults to `false`. | Boolean |
| `feature.convertToPickup.enabled` | Determines delayed order CTAs should be shown or not in Purchase history and Pickup instead button should be shown or not in order detail page. Defaults to `false`. | Boolean |
| `feature.lostInTransit.enabled` | Determines lostInTransit info on FC, LOT and ODP. Defaults to `false`. | Boolean |
| `account.referAFriend.isEnabled` | Determines whether the Refer a Friend option should be enabled on Account page. Defaults to `false`. | Boolean |

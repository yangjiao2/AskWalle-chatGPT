---
title: Glass Dashboard
navTitle: Glass Dashboard
---

## Getting Access to Splunk

You'll need to file a ticket with ServiceNow to get access to Splunk. You can file the ticket [here](https://walmartglobal.service-now.com/wm_sp/?id=wm_sc_cat_item_guide&sys_id=b3234c3b4fab8700e4cd49cf0310c7d7). You will need to open a ticket for each of the groups:
	- **Anivia Splunk Standard** - Access for anivia (devices index)
	- **CE-Splunk-Users** (readonlyUsers access group)
	- **CE_Splunk_Electrode_Users** (Users access group)
	- **CE_Splunk_CXO** - Access for cxo index

## Pulse Migration

If it's been a while since you've used Splunk, the schema changed in 2022.  Details of the migration and a list of the changed fields can be found [here](https://confluence.walmart.com/display/TEX/Glass+data+migration+for+Splunk+index%3Ddevices).

## MMS Migration

Some of the dashboards have been migrated to the [Managed Metrics Service](https://dx.walmart.com/documents/product/Managed%20Metrics%20Service/overview) to use [Grafana](https://grafana.com/docs/) and [Prometheus](https://prometheus.io/). MMS is much faster than Splunk, but has some tradeoffs that are important to note in deciding which tool to use. Here is a list of helpful links regarding the MMS migration specifically:
- Start here. Main MMS Migration document. How to write queries and dashboards in Grafana, overview of available metrics, and links to getting more help: [https://confluence.walmart.com/pages/viewpage.action?spaceKey=UAF&title=Mobile+-+Feature+Onboarding+into+MMS](https://confluence.walmart.com/pages/viewpage.action?spaceKey=UAF&title=Mobile+-+Feature+Onboarding+into+MMS)
- Join `#glass-mobile-mms-dashboard-migration`
- The syntax for writing queries is totally different and see these 2 docs for getting started: [https://promlabs.com/promql-cheat-sheet/](https://promlabs.com/promql-cheat-sheet/) and [https://prometheus.io/docs/prometheus/latest/querying/basics/](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- What is the difference between Splunk and MMS/Grafana? [https://confluence.walmart.com/pages/viewpage.action?spaceKey=UAF&title=Mobile+-+Feature+Onboarding+into+MMS]()
- [Data Pipeline Repo](https://gecgithub01.walmart.com/walmart-web/platform-metric-codec) - Main repo for creating the transformations for the data pipeline. One thing to note is that these are written in Java.
- [https://confluence.walmart.com/pages/viewpage.action?spaceKey=UAF&title=Feature+Onboarding+into+Pipeline+Development](https://confluence.walmart.com/pages/viewpage.action?spaceKey=UAF&title=Feature+Onboarding+into+Pipeline+Development) Onboarding to writing pipeline transformations and includes multiple Zoom recordings.
- [https://confluence.walmart.com/display/UAF/Mobile+Metrics+and+Cardinality](https://confluence.walmart.com/display/UAF/Mobile+Metrics+and+Cardinality) - The list of metrics and data available to you. See the repo for the most up to date list.

## MMS Dashboards

- [List of all MMS Dashboards](https://grafana.mms.walmart.net/dashboards/f/1890655824/customer-technology-customer-experience-ce-mobile-platform) - This is the list of all platform maintained dashboards including the ones below.
- [Network Health](https://grafana.mms.walmart.net/d/Ayps7eN4za2a2/wcp-native-network-health?orgId=1) - A list of all GraphQL calls across the whole app and the error count for each.
- [Mobile Errors Dashboard](https://grafana.mms.walmart.net/d/Y46QsgHVza2a2/mobile-error-dashboard) - Dig into errors for a specific feature.
- [Platform-Maintained Features](https://grafana.mms.walmart.net/d/ac1KntN4ka2a2/glass-platform-features-dashboard) - Dashboard covering all platform owned features.

# Splunk Dashboards

## General
- [Glass Launch Times](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/mobile_app_launch_times) - Launch times for iOS and Android with a distribution of the launch times.
- [Glass Score Card](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_score_card_clone)
- [Native Glass Mobile Operations View](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/native_glass_mobile_operations_view_clone) - General overview of app install, sessions, launch times, pageviews by feature, and other data.
- [Native Glass Page Performance - All Features](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/native_glass_page_performance__all_features_clone_apps) - Performance by page.
- [Performance Analytics - Post Tx](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/performance_analytics__post_tx_clone_apps) - Performance metrics for Account, Payment Methods
- [Glass CE Native | User WorkFlows | Summary](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_ce_native__user_workflows__summary_clone) - Number os requests, errors, and Performance for Homepage, MyItems, Search, Product Page, Cart, Checkout, and Account
- [Glass Mobile Nudge to Update](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_mobile_nudge) - Captures nudge updates by platform and other metrics related to users' action on nudge.

## Errors
- [Glass Diagnostic Message Dashboard](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_diagnostic_message_dashboard_clone) - Diagnostic message by team. All in `walmart-app-logs`
- [Glass Android Platform Diagnostic Message Details](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_android_platform_diagnostic_message_details_clone) - Diagnostic logs for Android
- [Glass Mobile App Errors - Used for App Rollout](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_mobile_app_errors__used_for_app_rollout_clone) - Shows top errors 
- [Mobile Error Drill Down](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/mobile_error_drill_down_clone_apps) - Top errors and the ability to drill down to a specific error.
- [Mobile Error rates](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/mobile_error_rates_clone_apps) - Error rates by context and top error messages
- [Customer Journey Explorer](https://ce-anivia-az.prod.us.walmart.net/en-US/app/search/glass__customer_explorer_pulse) - Track a customer's path through the entire app.

---

## App Shortcuts, Widgets, and Deep Links
- [Glass App Shortcuts](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_app_shortcuts_clone)
- [Glass Widgets](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_widgets) - Dashboard to track usage of iOS Widgets
- [Glass Mobile Deeplink](https://ce-anivia-az.prod.walmart.com/en-US/app/glass/glass_mobile_deeplink_dashboard) - Dashboard to track deep links attempted to route, success & failure in routing

## Bot Detection
- [Glass PerimeterX iOS Android](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_perimeterx_ios_android)

## Builds
- [Glass iOS Build Stats](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_ios_build_stats_clone) - Various stats about iOS builds including, build times, failing tests, app size, build steps and more.
- [Glass Android Build Stats](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_android_build_stats_clone_apps)

## Devices
- [Native Glass Mobile Operations View - Device Stats](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/native_glass_mobile_operations_view__device_stats_clone) - Stats by device OS and versions.
- [Mobile User Adoption Dashboard](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/mobile_app_adoptionplatform) - User adoption by app version

## Geofence
- [Glass Geofence Capability Dashboard](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_geofence_capability_dashboard_clone) - Multiple panels of Geofence enter/exit metrics

## Miscellaneous Features
- [Tabbar metrics](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/tabbar_metrics_clone) - iOS and Android tab bar usage metrics
- [Glass Onboarding Dashboard](https://ce-anivia-az.prod.walmart.com/en-US/app/glass/glass_onboarding_dashboard_clone) - Metrics related to the Onboarding feature.

## Network
- [Native Glass Mobile Operations - Network Health](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/native_glass_mobile_operations__network_health_clone) - Network response times
- [Glass OAOH iOS Service Response Breakdown](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_oaoh_ios_service_response_breakdown_clone) - API response times by statusCode.

## Notifications & Messaging
- [Glass Mobile Push Notifications Dashboard](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_mobile_push_notifications_dashboard_clone) - Stats related to push notifications
- [InAppMessaging Callout Diagnostic Dashboard](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_platform__inappmessaging_callout_diagnostic_dashboard_clone) - Glass Platform - InAppMessaging Callout Diagnostic Dashboard 

## Permissions
- [Glass Permissions iOS](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_permissions_ios) - App permission usage in Glass
- [Glass IDFA Soft Prompts](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_idfa_soft_prompts) - Dashboard to track IDFA soft prompts and metrics related to user's actions

## Scanner
- [Glass Base Scanner](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_base_scanner)
- [Glass Global Scanner](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/global_scanner_clone_pulse)
- [Product (Search) Scan Handler](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/global_scanner_search_handler) - Various metrics related to Scanner in search.

## User Impact
- [App_transactional_metrics Glass](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/app_transactional_metrics_glass_clone) - Compare GMV by versions
- [Feedback Results](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/feedback_results_clone) - User submitted feedback
- [Glass Auth User Synchronisation](https://ce-anivia-az.prod.us.walmart.net/en-US/app/devices/glass_auth_user_synchronisation) - Dashboard to track auth recorvery and user synchronisation 

## Non-Splunk Dashboards
- [NOC board 1](http://grafana.teflon.walmartlabs.com/d/Og_7q-8Gz/glass-monitoring-home?orgId=1) - Glass Monitoring
- [NOC board 2](http://crcgrafana.prod.walmart.com/d/BpA-YDkMz/oneops-change-monitor?orgId=1&refresh=1m&from=now-24h&to=now) - CRC Production Wall
- [NOC board 3](http://grafana.teflon.walmartlabs.com/d/aINrDx8Gk/glass-monitoring?orgId=1&from=now-24h&to=now&var-Platform=iphone) - Glass Monitoring
- [NOC Production Board used by NOC](http://crcgrafana.prod.walmart.com/d/wb-LA6sGz/glass?orgId=1&refresh=1m&from=now-12h&to=now) - Key Glass service dashboards

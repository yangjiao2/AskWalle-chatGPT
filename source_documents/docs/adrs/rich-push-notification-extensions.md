---
title: Rich Push Notifcation Architecture
navTitle: Rich Push Notifcation Architecture
---

## Introduction
Glass iOS US App currently supports marketing Rich Push Notifications via [Braze SDK(Appboy)](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/release/23.8/markets/usa/WalmartNotificationContent/NotificationViewController.swift#L9) provider.
Transactional text based push notifications are handled by our inhouse SmartComms/DMP team. Ask is to support Rich Push from SmartComms team(currently US market is the MVP), and long term goal is to support multi-markets.

## Motivation
Build native infrastructure to support Rich Push Notifcations from more than one provider. JIRA for ref - [CEPG-133311](https://jira.walmart.com/browse/CEPG-133311)

### Guiding Principles
- Must handle rich push notifications from different providers (Braze and SmartComms for US market).
- Needs to support multi-market in future.

## Finalized Approach: Option 1 
It aligns with our plugin architecture and will keep the extension targets lightweight.


## Considered Options

### Option 1: New RichPush Notification Plugin 

###### Description
New Plugin will:
- Handle notifcation payload parsing
- Create custom views for Rich Push display
- Networking to download images/videos
- Analytics

#### Pros:
- Any other market can reuse the plugin and configure it's own NotificationServiceExtension and NotificationContentExtension. Avoids code duplication.
- We don't bloat the extensions with a lot of logic and code.
- We can write unit tests for the logic within the Plugin. PS: No way to write unit tests for anything written in extensions.
- We can build the custom UI programatically and then embed the view inside xib in ContentExtension (no way to avoid this xib).

#### Cons:
- A little time-consuming to setup plugin, sample app, test targets, especially since it only needs to support the US market right now.


### Option 2: Implement the integration with SmartComms directly in NotificationServiceExtension and NotificationContentExtension.

###### Description
All the code related to notification payload parsing, networking and custom UI will reside in extensions for each market(right now only US).

#### Pros:
- Quick development since no additional plugin, test target and sample app setup needed. 

#### Cons:
- No code duplication right now, but possible when we have other markets are onboarded to use SmartComms RichPush.
- No unit tests since all the code will reside in Extensions.




### Note:
Irrespective of Option 1 or Option 2, each market will have:
- One NotificationService extension to support providers like Braze, Airship, SmartComms etc distinguished by an identifier in the payload.
- Each provider will have its own NotificationContent extension to support its own UI.

### Localization:
The notification payload will contain title, subtitle, body and action button titles and this content is dynamic in nature. Hence we cannot support localization for these fields on the app end. 
To support localization for action button titles, we will need to have pre-defined button texts and identifiers on the app side and match the identifiers to the payload's identifiers. But this approach will not allow the SmartComms team to send a new button identifier that is not supported by the app. 

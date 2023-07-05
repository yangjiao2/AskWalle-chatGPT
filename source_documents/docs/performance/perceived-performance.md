---
title: Perceived Performance
navTitle: Perceived Performance
---

# Perceived Performance

Spinners are one of the worst ways to tell the users our application is slow. We should move away from spinners from our main pages and move to perceived performance improvement screens. Perceived performance is how fast a page seems to the user. Perceived performance is used by multiple apps, Walmart Blue+ adopted perceived performance couple of years back after seeing a increase in trend of perceived performance pages on apps like facebook, etc which are content heavy. 


The main goal here is not block complete UI while network calls are being made to fetch content. We should rather show greyed out blocks (possible animation) while a network call is being made in the background. This helps freeing up other features which are not dependent on network call. 

<img src="https://gecgithub01.walmart.com/storage/user/29782/files/44156080-ff11-11ea-852b-3ff902552635" alt="Your image title" width="100"/> <img src="https://gecgithub01.walmart.com/storage/user/29782/files/41b30680-ff11-11ea-88ad-df3017c0fac9" alt="Your image title" width="100"/>
<img src="https://gecgithub01.walmart.com/storage/user/29782/files/5ba11900-ff12-11ea-995e-85d07ef47dd1" alt="Your image title" width="100"/><img src="https://gecgithub01.walmart.com/storage/user/29782/files/15988500-ff13-11ea-9976-60afc79e5fa3" alt="Your image title" width="100"/><img src="https://gecgithub01.walmart.com/storage/user/29782/files/19c4a280-ff13-11ea-895e-0965b78efdf5" alt="Your image title" width="100"/>


If you notice in the above blue+ and facebook images, the screen is immediately actionable after launch. The current glass app not so much. If the homepage API's are slow I will be waiting for homepage to be completely loaded. 

There should be two action items 

**Revisit the current pages and add perceived performance pages to them**

**Socialize this with Design & Product team to make sure this are thought about from start**

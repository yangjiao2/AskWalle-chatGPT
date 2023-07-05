---
title: Tempo
navTitle: Tempo
---

# Tempo

We have three main ways to support Tempo content in your screen.

1. The [response mapper](response-mapper.md) approach. This approach allows more customization at this point in time, but 
requires more work to integrate.
2. The [TempoUI](tempo-ui.md) approach. A more generic approach that relies on shared GraphQL fragments, 
shared UI and shared models as well as providing a view controller that can 
be used for a complete Tempo driven view controller. This is the recommended approach for new Tempo pages.
3. [TempoAPI](tempo-api.md) A reimagination of the response mapper to provide less duplication of GraphQL and association boilerplate.

In addition, there is a capability called [Tempo Preview](tempo-preview.md) which allows for testing of pre-production content. 

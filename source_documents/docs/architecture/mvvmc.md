---
title: Architecture - MVVM+C
navTitle: MVVM+C
---

# MVVM+C

The iOS Glass app uses the MVVM+C architecture pattern along with the [Combine](https://developer.apple.com/documentation/combine) framework. This decision is documented in the [iOS Architecture Pattern Evaluation](https://confluence.walmart.com/display/GPONEW/iOS+Architecture+Pattern+Evaluation).

- [Roles and Responsibilities](#roles-and-responsibilities)
- [Sample App](#sample-app)

## Roles and Responsibilities

### Network API

- Describes the (async) actions that can be performed on a remote system, including request and response models.
- Decouples Data Sources from specific backend services.
- If needed, provides default implementations and convenience extension methods.

_Note: Network APIs are typically implemented as protocols._

### Network Client

- Implements a **Network API** for a specific backend system.
- Stateless with respect to the request/response data.
- Maintains internal transport-level state, like HTTP cookies, if necessary.
- Encodes requests.
- Handles network communication.
- Supports environment-specific endpoint configuration.
- Decodes responses.
- If needed, defines internal "transport models" that map between request/response models of the **Network API** and actual on-the-wire data format.

### Data Source

- Acts as (local) source of truth for (some subset of) the application's state.
- Produces immutable representations of its current state.
- Fetches data from a **Network API** implementation.
- Queues and performs actions that mutate its state.
- Sends updates to a **Network API** implementation.
- Reconciles local state with updates from **Network API** results.
- Notifies subscribers of changes to its state.

### Coordinator

- Coordinates the UI for (some subset of) the application's flows.
- Subscribes to **Data Source(s)**.
- Instantiates **View Models** from the data from subscribed **Data Source(s)**.
- Provides **View Models** to its coordinated **Views**.
- Handles "actions" from its coordinated **Views** by:
	- Performing action(s) on **Data Source(s)**.
	- Invoking other **Coordinators**.
- Handles "results" of invoked **Coordinators**.

_Note: Coordinators are typically implemented as either a separate "plain" object, or as a top-level UIViewController._

### View Model

- Describes the valid states for (some subset of) the application's UI.
- Encapsulates all static state used by that subset of the applications UI.
- Converts from **Data Source** data into **View Model** instances, via initializer extensions (example).
- Provides formatting via computed properties/methods.
- Provides sub-models for descendant **Views**.

### View Delegate

- Describes the actions that a given **View** needs to invoke. (Actions that only affect internal, transient state do not need to be delegated.)
- Decouples **Views** from their surroundings.
- Performs the requested actions, or further delegates them.

_Note: View delegates are typically implemented as protocols, or on top of an eventing system._

### View

- Renders (some subset of) the application's UI
- Updates to reflect its current **View Model**.
- Maintains internal transient state for animations, edited form fields, expanded/collapsed sections, etc.
- Sends actions that aren't handled internally up to parent **Views** or **Coordinators**.

_Note: Views are typically implemented as a mixture of UIViews and UIViewControllers._

## Sample App

See the [Glass Sample App](https://gecgithub01.walmart.com/walmart-ios/glass-sample) as an example of how to apply this pattern.

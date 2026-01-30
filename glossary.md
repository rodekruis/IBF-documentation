### Definition of IBF

a system that displays and disseminates early warning notifications for an incoming disaster and visualizes relevant information to support decision making, following the country early action protocol.

### System components & definitions

The components of the **IBF System** are

* **IBF portal**, i.e. the frontend
* **IBF API**, i.e. the backend
* **IBF pipelines**, i.e. implementations of trigger models
* **IBF database**

### Main Concepts

* **disaster type**: IBF has separate forecasts and dashboard tabs for each disaster type: *floods*, *flash floods*, *typhoon*, *drought*, *malaria*. Also referred to as *hazard*.
* **event**: IBF pipeline can identify and upload 0 or more events in each pipeline run, where events can differ in terms of geographical location, *lead time*, *severity*, *exposure* and *probability*.
* **lead time**: Amount of time between forecast and forecasted start of *event*.
* **forecast**: Every pipeline run produces a forecast, which can be either an *alert* or *no alert*.
* **alert**: General term for either **warning** or **trigger**, meaning any event passing the lowest severity threshold set for that country and disaster type. Relates closely to **event**, the difference being that an event is (potentially) long-living across multiple pipeline runs, and can consist of multiple alerts.  
* **warning**: An *alert*/*event* that does not exceed *trigger* thresholds, but does exceed the (lowest) warning threshold (out of potentially multiple).
* **trigger**: An *alert*/*event* that exceeds the *trigger* threshold, which is typically set by *severity*, but alternatively by *lead time* or *accuracy*. A *trigger* can be forecasted by the pipeline, or it can be "set" by a user based on a forecasted warning.
* **set trigger**: A *trigger* that is manually set by a user based on a pipeline *warning*. In contrast to a pipeline-induced trigger.
* **notification**: An *alert* can or cannot lead to a notification, through either *email* or *WhatsApp*.
* **severity**: Measure of severity of a forecasted event. This measure directly relates to the different warning thresholds there might be. Is in principle separate from that event being a *trigger* or not.
* **exposure**: Measure of exposure of a forecasted event. There is a typically a **main exposure indicator** such as *population affected*, but any number of exposure indicators can be defined per country and disaster type.
* **probability**: Measure of probability of a forecasted event. E.g. operationalized by percentage of ensemble runs exceeding a certain severity threshold in the pipeline.
* **last upload time**: Last upload time from pipeline to API for a particular country and disaster type.
* **admin level**: Administrative level of administrative areas in a country, e.g. level 1 for Counties, 2 for Subcounties, 3 for Wards in Kenya.
* **admin area**: Particular administrative area, e.g. county 'Baringo' in Kenya. Having a *placeCode* as unique identifier, having geo-data on boundaries, and having any number of indicators with datapoints related to it.
* **event area**: The union of all *admin areas* in the *event*. Shown as such in National / multi-event view. (See [event areas](https://github.com/rodekruis/IBF-system/wiki/Features#3-event-areas))
* **early actions**: Actions that can be defined per country and disaster type that should be taken in case of an *alert*.
* **user role**: Role of a user, which determines what actions can be taken in the IBF Portal. (See [user roles](https://github.com/rodekruis/IBF-system/wiki/User-roles))
# Application Architecture

- [Application Architecture](#application-architecture)
    - [API Led Connectivity](#api-led-connectivity)
    - [Integration Patterns](#integration-patterns)

## Application Architecture
This section is a drill down of the Platform Architecture, covering the application related components and their interactions.

### API Led Connectivity

The key goal of the approach recommended by this document is to introduce reusable building blocks that can be reused both during the initial implementation and by future projects, resulting in reduced development effort.

API-led connectivity is a methodical way to connect data to applications through a series of reusable and purposeful modern APIs that are each developed to play a specific role – unlock data from systems, compose data into processes, or deliver an experience.

The API building block is a product that consists of functionality and simplicity required for the full lifecycle of APIs. This lifecycle consists of the ability to compose the data and connect to any other source of data. And it must provide full visibility, security, governance right from design.

The diagram below illustrates the API-led connectivity approach composed of three main layers:
- System APIs to unlock backend systems through a consistent contract, making use of our extensive connectivity, 
- Process APIs providing orchestration and transformation into business domain objects for greater agility and value creation and 
- Experience APIs focused on rendering information specific to devices or consuming client applications.

![alt text](./anypoint-platform-architecture/api_led.png)

The value of this approach is enabling a flexible, agile architecture built for reuse and consumption, to speed up project delivery with built in governance and security.

### Integration Patterns
Based on the discussions and discovery sessions, the following integration patterns were identified:

| Pattern | Description | Examples |
| --- | --- | --- |
| Real time, Synchronous | Real time or near real time requests to retrieve or process data, or trigger business processes, where the requester requires immediate feedback to support the scenario. | ● Address Verification <br> ● Policy Number Generation |
| Async - Fire and Forget | Async, near real time, Fire and Forget type scenarios where API clients send messages but do not need to wait for a response as part of their processes. | ● Policy Creation (async UW validation, core setup etc.) <br> ● Claim creation (rules processing, etc.) |

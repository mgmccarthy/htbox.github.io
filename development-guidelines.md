---
layout: default
---

# Development Guidelines

## Conventions and Patterns

### Naming Conventions for Constants

Please name constants like this:
`private const int EventToDuplicateId = 1;`

and not like this:
`const int EVENT_TO_DUPLICATE_ID = 1;`

### Unit Test Best Practices

#### Avoid Shared Test Setup 
exammple

#### Only Setup Data Needed For the Individual Test to Pass
exmaple 

### Unit Test Naming Convention

**Controllers**
Controller unit tests should be suffixed with "Tests"

**Handlers**
Handler unit test should be suffixed with "Should"


### Mediatr Components - Naming Conventions

The project has adopted a CQRS pattern using the Mediatr library. We have discussed ([read discussion](https://github.com/HTBox/allReady/issues/1262)) the naming for these components and agreed that we will *not* suffix them with "Async".

**Commands:**

A command message should end with "Command" and be named in an imperative style. e.g. A Command is an order, it tells something in the system to do a specific task. A Command usually carries more data on it than a Notification because commands are the main source of getting UI input into the system via handlers. A Command should have only one logical handler.

- EditCampaignCommand
- DeleteTaskCommand

The handlers should match the command message name with the suffix of "Handler". e.g.

- EditCampaignCommandHandler
- DeleteTaskCommandHandler

**Queries:**

A query message should end with "Query" and describe the data it returns. e.g.

- EventSummaryQuery
- TaskDetailQuery

The handlers should match the query message name with the suffix of "Handler". e.g.

- EventSummaryQueryHandler
- TaskDetailQueryHandler

**Notifications**

A notification message should end with "Notification" and describe the action that was performed. Notifications shoudl be named in past tense, and they inform subscribers that something has happened.

- CampaignEditedNotification
- TaskDeletedNotification

Since multiple subscribers could be interested in a given notification, it is possible for more than one handler to hanldle a Notification. Becaues of this, you may name Notification handlers whatever you think best describes their purpose when handling their given notification.

Unlike Commands, Notifications should not carry a lot of data on them. They should carry any type of unique identifiers (primary keys, etc...) that will be used by subscribers to correlate queries back to the database to get the infromation they need in order to do their job. This reduces temporal coupling of the system.

  
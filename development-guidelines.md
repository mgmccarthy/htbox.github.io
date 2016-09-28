---
layout: default
---

# Development Guidelines

## MediatR and CQRS

## MVC

### PRG

## Conventions and Patterns

### Naming Conventions

#### Constants

Please name constants like this:
`private const int EventToDuplicateId = 1;`

and not like this:
`const int EVENT_TO_DUPLICATE_ID = 1;`

#### Unit Tests

**Controllers**
Controller unit tests should be suffixed with "Tests"

**Handlers**
Handler unit test should be suffixed with "Should"

### Unit Test Best Practices

#### Avoid Shared Test Setup/Only Setup Data Needed for the Individual Test to Pass
Often, in the name of "reusability" a lot of programmers like to create shared test setup methods that are utlized by more than one unit test. Not only does this violate  a unit test's SRP  (each test should be self contained), but it can make refactoring those tests very difficult and often, leads to too much data being set up for a givne unit test to pass. Here's an example of shared test setup:

#### Pass in Null For Dependencies That Are Not Invoked For a Given Test

Just becaues a class takes three dependencies does not mean you need to provide a mock of each dependency to the class. Only dependencies that are being invoked for the given unit test need to pass in. Here is an example:

Given this Controller code:
```
[Area("Admin")]
[Authorize("OrgAdmin")]
public class EventController : Controller
{
    private readonly IImageService _imageService;
    private readonly IMediator _mediator;
    private readonly IValidateEventEditViewModels eventEditViewModelValidator;

    public EventController(IImageService imageService, IMediator mediator, IValidateEventEditViewModels eventEditViewModelValidator)
    {
        _imageService = imageService;
        _mediator = mediator;
        this.eventEditViewModelValidator = eventEditViewModelValidator;
    }

    // GET: Event/Details/5
    [HttpGet]
    [Route("Admin/Event/Details/{id}")]
    public async Task<IActionResult> Details(int id)
    {
        var campaignEvent = await _mediator.SendAsync(new EventDetailQuery { EventId = id });
        if (campaignEvent == null)
        {
            return NotFound();
        }

        if (!User.IsOrganizationAdmin(campaignEvent.OrganizationId))
        {
            return Unauthorized();
        }

        campaignEvent.ItinerariesDetailsUrl = GenerateItineraryDetailsTemplateUrl();

        return View(campaignEvent);
}
```

Here is a unit test that tests the Details controller action above returns NotFound when the Event it's trying to look up does not exist:

```
[Fact]
public async Task DetailsReturnsHttpNotFoundResult_WhenEventIsNull()
{
    var sut = new EventController(null, Mock.Of<IMediator>, null);
    var result = await sut.Details(It.IsAny<int>());

    Assert.IsType<NotFoundResult>(result);
}
```

A couple things to note here:
- null values are passed in for the `IImageService` and `IValidateEventEditViewModels` dependencies. This is because this unit test does not invoke those injected dependencies as part of what it's testing. 
- the minimal test setup. We only need to provide `Mock.Of<IMediator>` so we don't thrown a null reference exception when the meditaor is being invoked. By default, since we've done no set up on the mediator, it will return null from it's SendAsync method. As a result, the action method will return NotFound.
- we do not need to setup any type of Claims or mess around with IPrincipal at all because based on this specific test, we know this unit test will exit the code before we reach that execution point.
- we are passing `It.IsAny<int>` into the Details method as an argument instead of a made-up value... for example, "1". The reason is we don't care about that value for this unit test to test what it's being set up to do.  



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

  
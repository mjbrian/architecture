# Vertical Slice Arch

Works well with "Task-based" UI's

Model objects (domain entities) are used only within their respective "verticle"

## Query Reponses

CRUD Example... (didn't capture)
Complex Exmaple: Task-based UI
`Student Enrollment`
A Student can:

- Transfer
- Correct
- Schedule

Each Task will have an individual `Rquest` => `Handler`=> `Response`

- Transfer `Rquest` => `Handler`=> `Response`
- Correct  `Rquest` => `Handler`=> `Response`
- Schedule `Rquest` => `Handler`=> `Response`

## Command Reponses

_(Traditionally commands don't reuturn anything, `Command` => `Handler` => (`Void`))_

`Command` => `Handler` => `T`,
    Where `T` might be bool (did it succeed or not?)
    Where `T` might be an Id (302 Found, /orders/081-93-000)
    OverTime, `T` _can be_ refactored towards a `Command Result`, Where the object may look something like:

```C#
public class CommandResult<T>
{
    private CommandResult(string reason) => FailutreRason = reason;
    private CommandResult(T payload) => Payload = payload;

    public T Payload {get;}
    public string FailureReason {get;}
    public bool IsSuccess => FailureReason != null;

    public static CommandResult<T> Fail(string Reason) => new CommandResult<T>(reason);
    public static CommandResult<T> Success(T payload) => new CommandResult<T>(payload);

    public static implicit operator bool(CommandResult<T> result) => result.IsSuccess;
}
```

## Query Handlers

`Query` => `Handler` => `Response`

Handlers, enapulsate logic.
Examples.
`Query` => `EFCore` => `Response`
`Query` => `Dapper` => `Response`
`Query` => `HttpClient` => `Response`
`Query` => `MeatGrinder` => `Response`

Duplicate logic can happen. (where? between layers?)
Ask, Is it intentional or accidental duplication?

- if it is intentional duplication, pull that shared logic into something; a class, function, extension method.

- Don't intoduce any unnecessary abstractions

## Push behavior down

`Request` => `Handler`=> `Domain` => `Response`

- Request validation (has the user filled in all the inputs?)
- Push behavior down from the handler to the domain model
- Move logic away from the handler to the Domain
- Deep validation happens (ideally) in the domain
  - does this email already exist?
  - has this item already been purchased?

## Validation

### Request Validation

- Should be light weight
- Can only look at itself
- Can't reach out to another layer for validation

```C#
public class CoammandValidator : AbstractValidator<Command>
{
  public CommandValidator()
  {
    // https://fluentvalidation.net/
    RuleFor(m => m.LastName).NotNull().Length(0, 50);
    RuleFor(m => m.FirstMidName).NotNull().Length(0, 50);
    RuleFor(m => m.LastName).NotNull().Length(0, 50);
  }
}
```

### Domain Validation

Leans on Result Objects
This Ex, approves an invoice if the status is Approved

```C#
if(Status == Status.Approved)
  return CommandResult.Success;

if(Status == Status.Rejected)
  return CommandResult.Fail("Cannot approve a rejected order.");

Status = Status.Approved;
Send(new Order Approved
{
  Id = Guid.NewGuid(),
  OrderId = Id
});

return CommandResult.Success;
```

## Representations

Take the response and expose it to the outside world

`Query` => `Handler` => `Response` => ????

Web Api Example
/order
  /approve  =>  handler
  /reject   =>  handler
  /hold     =>  handler
  /cancel   =>  handler

handler => (returns) order.json => consumed by a `SPA`

1 response object (`order.json`) bound to a single view

What we're after:

- 1:1 response obj w/ representation(s)

handlers may share duplicate logic, as a remedy see decorator pattern (dubbed 'pipeline behavior')

The 'pipeline behavior' can:

- wrap the handler
- stack with other pipeline behaviors
- similar to action filters in C#

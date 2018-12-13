# Refactoring from an Anemic Domain Model to a Rich Domain Model
-----------------------------------------------------------------------------------------------------
# Anemic domain model:
* Entities with no behavior, only data properties
* All behavior inside services, controllers or ui layer
* Lacks encapsulation (cannot guarantee invariables, meaning valid state of entities, mutable parts are accessible from everywhere and open to side-effects)
* Lacks explorability (code cannot be easily explored because data and code that works on that data are in separate places)
* It is in procedural style (transaction script pattern)
* Procedural style code does not meain it is in functional style because in functional style, objects should be immutable and pure functions can not change the state of an object
-----------------------------------------------------------------------------------------------------
# Domain model exposed to outside through an api contract:
* Entities are matched to db tables and columns, and also mostly exposed to outside in the api contract, so refactoring entities breaks  the db and the api contract
* Prone to over-posting attacks (posting internal fields which should have been encapsulated but are exposed)
* Use DTOs instead of exposing the domain models (entities, value objects and aggregates)
* Do not reuse the same DTO for get and post api methods to prevent over-posting attacks (use ...DTO, Create...DTO etc.)
* Do not reuse the same DTO for different get api methods or different post api methods (use Create...DTO, Update...DTO, Process...DTO, etc.)
* Do not reuse the same DTO for single get and list api methods (in get list api methods use ...InListDTO with less fields and nesting)
* Use manual mapping instead of using AutoMapper to guarantee api contract does not change when refactoring the domain model
* Do not put validation attributes on DTO properties, move validation logic inside domain models and reuse this validation logic inside the controller code when creating entities from the DTO
* Do not expose enum types, instead expose string because enums are an implemntation detail and later can be changed to some other type
* DTOs can be thought of as an Anti-corruption layer in DDD, can be used both for outside facing apis, microservice apis, or for apis surrounding any bounded context
-----------------------------------------------------------------------------------------------------
# Moving some of the validation and business logic into value objects:
* Value objects are immutable, simple to track because they are not prune to side-effects
* Value objects are only identified and differentiated by their values, they do not have an id
* Value objects have encapsulated value fields and behaviors that work on these fields
* Value objects can have primitive type fields or other value objects that change together and are verified together, or can have even a single value field
* Single field value object types (like CustomerName and Email, or Dollars and Distance which are strings/decimals but have different verification rules), can have a get only Value property
* Make constructor private and use a static Create() factory method to build a valid value object, return type should contain both the value object created and the validation errors if any
* Do not introduce an unnecessary value object type if there is no validation or logic to do with this value, just use the primitive type instead
* Make the property setters private or protected (for the ORM to work)
-----------------------------------------------------------------------------------------------------
# Moving rest of the logic into entites:
* Entities keep mutable state, have an id, and are identified by this id property
* Entities, like the value objects, have encapsulated value fields and behaviors that work on these fields
* Do not expose collections (Lists, IEnumerables) directly to the outside from the entities
* IEnumerable and IQueryable are lazy evauated, so use them carefully
* When needed, expose a public getter only property of type IReadonlyList instead and return a new clone of the private collection in this getter
* Move as much of the remaining logic, that is not covered in the value objects, into entities
* Entitites should have cohesive behavior (operation should really belong to this entity), and satisty single responsibility principle
* Move behavior which does not fit in the entity to other classes or keep this behavior in services
* Make constructor private and use a static Create() factory method to build a valid entity, return type should contain both the entity  created and the validation errors if any
* Do not put both the entity class reference and the foreign key id of that referenced entity, drop the foreign key property or the class reference property
* You may think of using the aggregate pattern like in DDD when keeping class reference to other entities, else the entity can become more complex than necessary for its transactional behavior
* Make the static factory method (Create(), Create...() internal if this entity is not an aggregate root (because only the aggregate root can create a new instance of its composing entities)
* Make the property setters private or protected (for the ORM to work)
* Sometimes it is better to move code that changes state and code that return a result into seapate methods to simplify the api of the entity (command query separation), like string CanProcess(...) and void Process(...) instead of string Process(...) or Result<...> Process(...)
-----------------------------------------------------------------------------------------------------
# Moving code from fat controllers into application services and middleware:
* Application service is not a domain service like in DDD, and does not have domain logic in it, but just controller logic
* If there is too much code in a controller, then it is better to treat the controller as a facade and delegate work to smaller more cohesive application services
* If we are not using the aggregate pattern in DDD (where aggregate represents a business transaction), then the repositories do npot work on only aggregates and work with any entity and should not commit to db, and should not have a SaveChanges() method, and the responsibility of committing unit of work (...Context in Entity Framewok) to db belongs to the web request method on the controller, and it can be centralized in a base controller class so that after successful invocation, it commits (calles unit of work's SaveChanges() automatically
* try catch blocks returning http 500 in an exception can delegate this responsibility into a custom exception handling middleware in asp.net core or exception handler in webapi
-----------------------------------------------------------------------------------------------------
# Solution/project structure and dependency guidelines:
* Do not share domain logic code between different bounded contexts, coupling is worse than code duplication, apply DRY only inside a bounded context
* Do not create generic domain libraries, apply YAGNI, write minimum amount of code to solve the specific business problem, code and tests are a liability
* Do not include any infrastructure or external dependencies in the domain models, handle them inside application services
* Do not inject external services into the domain model, call the outside api from the application service and pass the resulting data into the domain model instead
* Domain models should have 100% test coverage
* Use folders resembling domain concepts or features instead of folders resembling class types (like controllers, views, repositories, etc.)

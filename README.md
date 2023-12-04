# Self-Service Active Directory Application

## _The good way account management on Active Directory_

Self-Service AD (SSAD) is a web-based password management solution. It eliminates the users dependency on administrators to change or reset their passwords. It reduces the workload of the help desk and in turn reduces the cost incurred by the company.

Every day help desk and IT personnel waste time handling minor problems that regular users can solve by themselves. Every password reset, create user, user info update, enable/disable user in Active Directory involves skilled personnel whereas users can easily perform these tasks for themselves. Providing users with the possibility of Active Directory self-service saves human resources and streamlines updating of the active directory information. This app helps your Core IT Staff focus on the daily challenges to run and improve your Environment.

This application used in 9 different organizations, the password reset workload of IT personnel has decreased by 65% with the self-service password reset feature in the last 3 years.


## How does it work?

In cases where the user does not have an active directory account, the phone number is not registered or is not up to date, or the account is inactive, it automatically creates a ticket to the IT department and if the IT department approves the ticket, it automatically performs all the necessary active directory user transactions and sends the information to the user's phone via SMS. 

With automatically created tickets, IT employees can perform transactions with the tickets in the portal without having to perform manual transactions with the active Directory console or 3rd party Active Directory applications.


## Features

What can IT departments do?
- Can create and reset accounts through the administration panel.
- Can inspect organizational units and user details.
- Can enable/disable/lock and unlock users.
- Can list all user actions via portal.
- Can list locked users,password expired users and never password expires users.
- Can manage profile update requests.
- Access can be restricted for the unwanted user.
- System administrators can create authorized delegate users for certain organizational units for IT department employees. Thus, delegated users can only create and view users within the organizational units for which they are authorized.
- Can store and view the contracts signed by users by uploading them to the portal.

What can users do?
- It can perform password reset, password change,password check,disable user and account opening operations via portal.

**How self-service password reset works**
- A user who has forgotten their password initiates a password reset request from either the Self-Service AD web portal.
- Self-Service AD checks the LDAP user status.
- After successful identity verification, a random password is generated according to the predetermined password criteria and sent to the registered mobile phone number of the user.
- The user will then be able to log in to their account using the newly reset password.
  

### Advantages of Self-Service AD application

:heavy_check_mark: Does not require a server installation that is a member of the Active Directory domain.
:heavy_check_mark: The delegate user does not need to be a member of the Domain Admin group. For delegation, it is sufficient to define a delegation user with limited authority to the OUs and groups that the portal can process.
:heavy_check_mark: Since it has a distributed architecture, projects can be located on different servers (UI, API, Background services).

### Prerequisites

```sh
Elastic APM
Microsoft Active Directory
MongoDB
RabbitMQ
Redis
SQL Server
```

## Project structure

Project  | Technology  | Type | Description
------------- | ------------- | ------------- | -------------
SelfServiceAD.Business  | .NET 7.0  | Class Library | Business layer
SelfServiceAD.Caching  | .NET 7.0  | Class Library | Caching layer
SelfServiceAD.Core  | .NET 7.0  | Class Library | Core layer
SelfServiceAD.Data  | .NET 7.0  | Class Library | Data access layer
SelfServiceAD.Entities | .NET 7.0 | Class Library | Entities
SelfServiceAD.DhcpServices | .NET 7.0  | Worker Services | Dhcp server communication layer
SelfServiceAD.LdapServices | .NET 7.0  | Class Library | LDAP communication layer
SelfServiceAD.LoggingServices | .NET 7.0  | Class Library | Logging services
SelfServiceAD.Messaging | .NET 7.0  | Class Library | Message broker services
SelfServiceAD.NotificationServices  | .NET 7.0 | Worker Services | SMS,Slack,Email,Logging services
SelfServiceAD.HealthCheck  | .NET 6.0 | Web Application | Healtcheck API
SelfServiceAD.WebAPI | .NET 7.0 | Web Application | All API endpoints for UI
SelfServiceAD.WebUI  | .NET 7.0 | Web Application | Razor pages

## Database structure

Description  | Database type  | Purpose
------------- | ------------- | -------------
Application database | MS SQL Server  | Stores application data,identity information and active directory organizational unit and group paths and other application-related details
Hangfire database| MS SQL Server  | Stores Hangfire server data
Log database | MongoDB | Stores application log data


## Layers

### Business Layer

The Business project provides a large number of classes that handle business rules, validations, LDAP calls, message queue operations, constants, option definitions, and caching operations used throughout the application. It also provides CRUD operations by calling the repositories in the Data layer.

**System.DirectoryServices** and **System.DirectoryServices.AccountManagement** libraries are used for active directory account management.

The `JsonStringLocalizer` helper methods in the Core project and the Business layer throughout the application contain the Response class, which is the common return model of WebAPI endpoints.

Example response class:

```
{
    "data": [
        {
            "companyId": 73,
            "description": "Company A"
        },
        {
            "companyId": 74,
            "description": "Company B"
        }
    ],
    "statusCode": 200,
    "resultCode": 0,
    "isSuccess": true,
    "message": null,
    "hasData": true
}
```

### Data access layer

In the Data project, the necessary contexts have been written to access the MSSQL database using Repository pattern and EntityFramework, and the MongoDB database with MongoClient. UnitOfWork pattern is used for SQL database. Sample data is added to the empty database with the seeding classes in the `/Migrations` directory.

### Entities layer

DTO, Model and Entities were written in the Entities project. Each class implements an abstract class.

Defined interfaces|
-------------|
IModel |
IDto |
ISqlBaseEntity|
INoSqlBaseEntity|

### WebUI and WebAPI projects

The WebUI project communicates with the SQL database and the endpoints in the WebAPI project. Classes that communicate with API services make HTTP client requests through the classes added to the `/ApiServices` directory of the WebUI project. A separate ApiService class is written for each controller.

API endpoints authenticate with JWT. It is configured for JWT authentication with the `/Extensions/AddIdentityExtension` method. Some endpoints related to background services require authentication with API key. (Example: User export operations, Caching services..) `ApiKey` methods in `/Attributes` directory are ApiKey validation at Controller level. (Example: `CacheController,NotificationsController,UserImportsController` ). API key definitions are stored in the `appSettings.json` file. (Example: NotificationApiKey, UserExportApiKey)

Localization related settings are defined in `/Core/Helpers/JsonStringLocalizer` and `/Core/Helpers/JsonStringLocalizerFactory` classes.

The necessary settings for CORS configuration have been added to the `Program.cs` file as middleware with the `/Extensions/AddCorsOptionExtensions` extension method.

It has been added to the `Program.cs` file as middleware with the `/Extensions/AddSwaggerExtensions` and `/Extensions/AddSwaggerAuthorized` extension methods for the Swagger implementation. The swagger user and password are set in the 'SwaggerOptions' node of the `appSettings.json` file.

FluentAPI validations for model validations are written in classes added to the `Business/ValidationRules/FluentValidation` directory. Validations within WebAPI and WebUI projects have been added to the `Program.cs` file as middleware with the `/Extensions/AddValidatorsExtensions` extension method.

Validations in `/WebAPI/` and `/WebUI/` projects are written in object mapping classes added to the `/Mapping/` directory.

The logging operations are written in the `WebUI/Filters/TimerFilter` class to monitor the action performances. Detailed information such as the execution time of the action performances are sent to the RabbitMQ message queue with this class. The logging messages are read and written to MongoDB by the background service worker named `LoggingServices`.

Database contexts used for WebAPI and WebUI projects have been added to the Program.cs file as middleware with the `/Extensions/AddDbContextsExtensions` extension method.

Classes related to Hangfire implementations have been added to the Program.cs file as middleware with the `/Extensions/AddJobsExtensions` extension method in the WebUI project.

In order to map the settings used in the application from the `appSettings.json` file, the classes in the `Business/Abstract/Options` and `Business/Concrete/Options` directory are written. By using [Options pattern](https://docs.microsoft.com/en-us/dotnet/core/extensions/options), the records in the `appSettings.json` file are structured and implemented with Dependency Injection.

RateLimiting related settings are defined in `ClientRateLimiting` node of `appSettings.json` file.

ElasticAPM is used to monitor the performance of applications. ElasticApm related settings are defined in the `ElasticApm` node of the `appSettings.json` file.

In the WebAPI project, the '/Extensions/IpSafeMiddleware' class has been added as middleware to the 'Program.cs' file so that only certain IP addresses in the 'IpOptions' node of the 'appSettings.json' file can access the API.

The JWT settings used for authentication are in the `appSettings.json` file `TokenOptions` node.

Serilog-related settings are defined in the `Serilog` node of the `appSettings.json` file for dynamic logging. By default, File logging and MongoDB sinks are defined.

**Redis** is preferred as the cache technology in the application architecture. The data required for the control of active directory users are used by keeping them in the cache.

JSON files are used to store localized strings and implement middleware to switch languages ​​via language keys in the header. Via IDistributedCache, strings are retrieved from the cache with the JsonStringLocalizer class, which implements the IStringLocalizer abstract class. Strings are created in JSON format in the /Resources/ directory of WebAPI and WebUI projects. The language in which the application will start broadcasting as soon as it stands up is defined in the `Language` node of the `appSettings.json` file.
In the WebUI project, the language settings can be changed dynamically with the help of the `ViewComponents/LanguageSwitcherViewComponent` class by selecting them from the drop down list added to the layouts. Static MVC pages are displayed in the selected language.
In addition, after the language selection made in the WebUI project, the `Accept-Language` header information for API requests is automatically set with the help of API call classes in the `/ApiServices` directory.

### Notification services

In the NotificationServices project, the services required for sending SMS, Slack and E-mail are written.  [Twilio](https://www.twilio.com/),[Teknomart](http://www.teknomart.com.tr/),[NetGSM](https://www.netgsm.com).Also, Generic SMS service has been written for companies that support HTTP GET method.

**RabbitMQ** is preferred as the message broker in the application architecture. Asynchronous endpoints such as sending SMS are made through services that process RabbitMQ message queues. 

The application deserializes the pending notifications in the message queue and sends the incoming object via the relevant service. Since the messages waiting in the message queu are of a generic nature, a service can be started from Twilio by standing up for the Twilio service at the time of T, or sending from the Generic sms service can be started by standing up for the Generic sms service. After the services are running, they are sent to the logging queue and written to MongoDB by the background service named NotificationServices.


### Entities

|Table   | Description  |
|---|---|
|Companies   |Company names are stored for company contracts.|
|CompanyContractFiles|This is the table where company agreement files are stored.   |
|CompanyContracts|This is the table where company agreements are stored.   |
|Groups   |Is is table where the groups of IT/helpdesk users are stored.|
|MacAddressFilterRequests   |It is the table where mac address filter requests are stored.|
|NotificationTemplates   |It is the table where message templates are stored to enable IT/help desk personnel to send specific sms texts to users by creating templates outside of application functions.|
|OrganizationalUnits   |Is is table where the organizational units of IT/helpdesk users are stored.|
|Organizations   |It is the table where the organization information is stored in order to determine from which organization the user requests sent through the portal are sent and to determine on which group the IT / help desk personnel will create the user.|
|Titles   |While creating the active directory user account, the values that can be written in the title attribute field are stored.|
|UserContractFiles   |This is the table where user agreement files are stored.   |
|UserContracts   |This is the table where user agreements are stored.   |
|UserGroups   |It is the table where the group privileges of IT/helpdesk users are stored. It is used to determine which group can be made a member of users who do not have the administrator role but have the IdentityManager authority.|
|UserKpsServiceParameters   |It is the table where KPS(See: Self-ServiceAD.WebAPI project KpsServicesController.cs file) user passwords are stored.|
|UserOrganizationalUnits   |Is is table where the organizational unit entitlements of IT/helpdesk users are stored. It is used to determine in which organizational units users who do not have the administrator role but have the IdentityManager authority can create users.|
|UserRequests   |It is the table where user account creation, information update, account creation requests are stored.  |
|UserJwtTokens   |It is used to store JWT tokens. A request is sent to the Login/Access Token endpoint in the Self-Service.WebAPI project with the POST method.|

### Predefined roles

```sh
Administrator
CompanyContractEditor
CompanyContractManager
ComputerViewer
CreateUserWithoutRequest
DhcpRequestManager
DhcpRequestSender
IdentityManager
KpsUser
MailSender
NotificationSender
OrganizationalUnitViewer
SlackMessageSender
SmsSender
UserContractEditor
UserContractManager
UserRequestManager
```


### Predefined user check rules and result codes for password reset

Defined in `SelfServiceAD.Entities/Enums/UserRequestStatusCode` 

|User check enums(1xxx)   | Code  |
|---|---|
|UserIsNotFound   |1001|
|IdentityValidateError   |1002|
|UserPhoneNotFound   |1003|
|UserPhoneIsNotMatched   |1004|
|UserPhoneNotFoundAlsoNotActive   |1005|
|UserIsNotActive   |1006|
|UserNotActiveAlsoPhoneNotMatched   |1007|
|DuplicateApplication   |1008|
|AllUserValidationsPassed   |1009|
|UserAlreadyDisabled   |1010|
|UserIsNotExist   |1011|



|User save enums for tickets(2xxx)   | Code  |
|---|---|
|UserIsNotFoundSaved   |2001|
|AllUserValidationsPassedSaved   |2002|
|UserPhoneNotFoundAlsoNotActiveSaved   |2003|
|UserNotActiveAlsoPhoneNotMatchedSaved   |2004|
|UserIsNotActiveSaved   |2005|
|UserPhoneNotFoundSaved   |2006|
|UserPhoneIsNotMatchedSaved   |2007|
|DuplicateApplicationSaved   |2009|
|UserDisableRequestSaved   |2010|

|User request result statuses(3xxx,4xxx)   | Code  |
|---|---|
|UserCreatedOrUpdated   |3001|
|UserActivatedPhoneNumberSaved   |3002|
|UserActivatedPhoneNumberUpdated   |3003|
|UserActivated   |3004|
|PhoneNumberUpdated   |3005|
|PhoneNumberSaved   |3006|
|UserRequestDeleted   |3007|
|UserDisableRequestApproved   |4007|
|UserCreatedWithoutRequest   |4008|


## Technology stack

Self Service Active Directory Application uses a number of projects to work properly:

- [AspNetCore.HealthChecks.MongoDb](https://www.nuget.org/packages/AspNetCore.HealthChecks.MongoDb/)
- [AspNetCore.HealthChecks.Rabbitmq](https://www.nuget.org/packages/AspNetCore.HealthChecks.Rabbitmq/)
- [AspNetCore.HealthChecks.Redis](https://www.nuget.org/packages/AspNetCore.HealthChecks.Redis/)
- [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/)
- [AspNetCore.HealthChecks.System](https://www.nuget.org/packages/AspNetCore.HealthChecks.System/)
- [AspNetCore.HealthChecks.UI](https://www.nuget.org/packages/AspNetCore.HealthChecks.UI/)
- [AspNetCore.HealthChecks.UI.Client](https://www.nuget.org/packages/AspNetCore.HealthChecks.UI.Client/)
- [AspNetCore.HealthChecks.UI.InMemory.Storage](https://www.nuget.org/packages/AspNetCore.HealthChecks.UI.InMemory.Storage/)
- [AspNetCoreRateLimit](https://www.nuget.org/packages/AspNetCoreRateLimit/)
- [AutoMapper.Extensions.Microsoft.DependencyInjection](https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection/)
- [Elastic.Apm.NetCoreAll](https://www.nuget.org/packages/Elastic.Apm.NetCoreAll/)
- [FluentValidation](https://www.nuget.org/packages/FluentValidation/)
- [Hangfire.Core](https://www.nuget.org/packages/Hangfire.Core/1.7.28)
- [jQuery](https://www.nuget.org/packages/jQuery/)
- [Microsoft.AspNetCore.Authentication.JwtBearer](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/6.0.5)
- [Microsoft.AspNetCore.Identity](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/)
- [Microsoft.AspNetCore.Identity.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity.EntityFrameworkCore/6.0.5)
- [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/6.0.5)
- [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/6.0.5)
- [MongoDB.Driver](https://www.nuget.org/packages/MongoDB.Driver/)
- [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/)
- [RabbitMQ.Client](https://www.nuget.org/packages/RabbitMQ.Client/)
- [RestSharp](https://www.nuget.org/packages/RestSharp/107.3.0)
- [Serilog](https://www.nuget.org/packages/Serilog/2.11.0)
- [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/)
- [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/)
- [System.DirectoryServices](https://www.nuget.org/packages/System.DirectoryServices/6.0.0)
- [System.DirectoryServices.AccountManagement](https://www.nuget.org/packages/System.DirectoryServices.AccountManagement/6.0.0)

## Running projects (with Docker)

Each required servers contains the docker-compose.yaml which describes the configuration of required services. All containers can be run in a local environment by going into the root directory of each one and executing:

```sh
docker compose up
```

Edit environment options on ${appsettings.EnvironmentName.json}, select environment (Development, Staging or Production) and then execute projects.



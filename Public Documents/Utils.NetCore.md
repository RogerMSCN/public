# Utils for .net core 5.0
## Summary
This is a collection of utils based on .net core 5.0. It will help you to manage a few normal fuctions:
- Write log.
  - Write log to log file.
  - Write log to a sql server database.
  - Write log to a dynamics 365 environment.
- Send request to dynamics 365 web api.

For these utils, you need to create user settings in default configuration file named "appsettings.json" with root section named "UserSettings", i.e.
```json
  "UserSettings": {
    /*
        Use user secrets to store secrets.
        You can also use azure key vault or secret file.
        If used azure key vault, the value of SecretStorageMedia is "Azure Key Vault".
        If used user secrets, the value of SecretStorageMedia is "User Secrets".
        If used secret file, the value of SecretStorageMedia is "Secret File".
        If does not set the value of SecretStorageMedia, it will use azure key vault by default.
    */
    "SecretStorageMedia": "User Secrets",
    /* If the value of SecretStorageMedia was "User Secrets", must set the value of UserSecretsId. */
    "UserSecretsId": "00000000-0000-0000-0000-000000000000",    
    /*
        If the value of SecretStorageMedia was "Secret File", must set the value of SecretFileName.
        Notes: secret file must be a json file.
    */
    "SecretFileName": "MySecretFile.json",
    /* Settings for logger */
    "LogSettings": {
      /* If true, will write log to a log file. */
      "FileLogEnabled": true,
      /* If true, will write log to a database. It will write to a sql server database. */
      "DatabaseLogEnabled": true,      
      /* If true, will write log to a dynamics 365 environment. */
      "Dynamics365LogEnabled": true,
      /* Settings for file logger. */
      "FileLogSettings": {
        /* 
            Date pattern in log file name. 
            The full log file name is [Host Name of the local computer]-[Date].log
        */
        "LogFileNameDateTimePattern": "yyyyMMdd",
        /* 
            Defines the lowest logging severity levels.
            Log level includes:
            Trace = 0
            Debug = 1
            Information = 2
            Warning = 3
            Error = 4
            Critical = 5
            None = 6
            All log with level higher than MinLogLevel will be written.
        */
        "MinLogLevel": "Trace",
        /* Root path of log file. */
        "RootPath": "./bin/logs",
        /* If log file size is more than MaxFileSizeInMB, will create a new log file. */
        "MaxFileSizeInMB": 10
      },
      /* Settings for dynamics 365 environment which log will be written to. */
      "Dynamics365LogSettings": {
        /* The name of client id in secret storage. */
        "ClientIdName": "My Client Id",
        /* The name of client secret in secret storage. */
        "ClientSecretName": "My Client Secret",
        /* The name of tenant id in secret storage. */
        "TenantIdName": "My Tenant Id",
        /* The root url of dynamics 365. */
        "Resource": "https://myorg.crm.dynamics.com",
        /* The prefix of publisher which is used to publish the solution. */
        "PublisherPrefix": "new",
        /* Service Root URL of dynamics 365 web api. */
        "WebApiRootUrl": "https://myorg.api.crm.dynamics.com/api/data/v9.2",
        /* The name of log entity in dynamics 365, includes prefix. */
        "EntityName": "new_logdetails",
        /* Defines the lowest logging severity levels. */
        "MinLogLevel": "Trace",
        /* 
            The field name will be defined by default, if you needed to define them by yourselves, please set here.
            The fields include:
            LogTimeFieldName
            CategoryFieldName
            LevelFieldName
            MessageFieldName
            ExceptionMessageFieldName
            StackTraceFieldName
            LoggerTypeFieldName
            StartTimeFieldName
            EndTimeFieldName
            CostTimeInSecondsFieldName
            ActionIdentifierFieldName
            UserIdFieldName
            UserLoginNameFieldName
            ActionFieldName
            WebApiLogTypeFieldName
            InContentFieldName
            OutContentFieldName
        */
        "CategoryFieldName": "name",
        /* 
            The type of authentication provider. Includes:
            AzureApp
            AzureUser
            AzureApp, as an application registered in azure active directory. If this, ignore user id and user password. The grant type in token request is client_credentials.
            AzureUser, as a user in azure active directory, If this, must input user id and user password. The grant type in token request is password. When gets a valid token, will get a valid refresh_token too, may get new token by this refresh_token.
        */
        "AuthenticationType": "AzureUser",
        /* Domain name. */        
        "Domain": "MyDomain",
        /* The name of user id in secret storage. */
        "UserIdName": "User Id",
        /* The name of user password in secret storage. */
        "UserPassName": "User Pass",
        /* The root url of active directory federation services. */
        "ADFSRootUrl": "http://myorg/api/data/v9.0"
      },
      /* Settings for sql server which log will be written to. */
      "SqlDbLogSettings": {
        /* SQL Server address, it may be an IP address too. */
        "DataSource": "myserver.database.windows.net",
        /* Default database name. */
        "InitialCatalog": "MyDatabase",
        /* If true, need to set sql user id and sql user password. */
        "UserCredentialNeeded": true,
        /* 
            The name of sql user id in secret storage. 
            If UserCredentialNeeded was true, can not empty. 
        */
        "UserIdName": "SQL User Id",
        /* 
            The name of sql user password in secret storage. 
            If UserCredentialNeeded was true, can not empty. 
        */
        "UserPassName": "SQL User Password",
        /* The name of store procedure used to write log. */
        "StoreProcedureName": "MyStoreProcedure",
        /* Defines the lowest logging severity levels. */
        "MinLogLevel": "Trace"
      }
    }
  }
```
## Write log
If you wanted to write log, please register dependency first.
#### Asp.Net Core
Register logger provider in ***Program.cs***
```c#
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            })
            .ConfigureLogging(logging =>
            {
                logging.ClearProviders();
                /* 
                    If you only wanted to write log to dynamics 365, please only register dynamics 365 logger provider.
                */
                //logging.AddDynamics365Logger();
                /* 
                    If you only wanted to write log to log file, please only register file logger provider.
                */
                //logging.AddFileLogger();
                /* 
                    If you only wanted to write log to sql server, please only register sql db logger provider.
                */
                //logging.AddSqlDbLogger();
                /* 
                    If you wanted to write log to several environments, please register full logger provider, the flag of log enabled will control whether you can write log to that environment.
                */
                logging.AddFullLogger();
            });
```
Register necessary components in ***Startup.cs***
```C#
    public void ConfigureServices(IServiceCollection services)
    {
        /* Add http client component for http request. */
        services.AddHttpClient();
        /* Add the type declare, it will be used to load secrets from secret storage. */
        services.AddSingleton<ISecretConfiguration, SecretConfiguration>();
        /* 
            Add the settings declare, it will get all settings from app configurations, UserSettings.Name is the name of root section in app configurations. 
        */
        services.Configure<UserSettings>(Configuration.GetSection(UserSettings.Name));
        services.AddControllers();
    }
```
In any models which include logger, create logger in constructor.
```C#
    private readonly ILog<WeatherForecastController> _logger;
    public WeatherForecastController(ILog<WeatherForecastController> logger)
    {
        _logger = logger;
    }
```
Then call functions to write log.
```C#
    _logger.LogInformationAsync(DateTime.Now.AddDays(-1), DateTime.Now, "Test LogInformation");
```
#### Application Core
Register dependencies in entry.
```C#
    IConfiguration config = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .Add(new JsonConfigurationSource { Path = "appsettings.json", ReloadOnChange = true })
        .Build();

    var services = new ServiceCollection()
        .AddScoped<ITestOutput, TestOutput>()
        .AddSingleton<ISecretConfiguration, SecretConfiguration>()
        .AddLogging()
        .AddHttpClient()
        .AddFullLogger()
        .AddOptions()
        .Configure<UserSettings>(config.GetSection(UserSettings.Name))
        .BuildServiceProvider();
```
#### Dynamics 365 implement
Create a log entity, includes fields needed.
- Table name: new_logdetail
- Columns:
  - new_action
  - new_actionidentifier
  - new_costtime
  - new_endtime
  - new_exceptionmessage
  - new_incontent
  - new_level
  - new_logdetailid
  - new_loggertype
  - new_logtime
  - new_message
  - new_name
  - new_outcontent
  - new_stacktrace
  - new_starttime
  - new_user
  - new_userloginname
  - new_webapilogtype

[Find more details](https://github.com/RogerMSCN/public/blob/main/Public%20Documents/LogSolution.xlsx).
#### SQL Server implement
Create table.
```SQL Script
    Create Table LogDetail
    (
        [ID] uniqueidentifier default NEWSEQUENTIALID() not null primary key,
        [LogTime] datetime null,
        [Category] varchar(500) null,
        [Level] varchar(100) null,
        [Message] varchar(max) null,
        [ExceptionMessage] varchar(max) null,
        [StackTrace] varchar(max) null,
        [LoggerType] varchar(100) null,
        [StartTime] datetime null,
        [EndTime] datetime null,
        [CostTime] decimal(34,5) null,
        [ActionIdentifier] varchar(500) null,
        [UserId] varchar(100) null,
        [UserLoginName] varchar(500) null,
        [Action] varchar(500) null,
        [WebApiLogType] varchar(100) null,
        [InContent] varchar(max) null,
        [OutContent] varchar(max) null
    )
```
Create store procedure.
```SQL Script
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO

    CREATE PROCEDURE [usp_InsertLog]
    (
        @LogTime datetime,
        @Category varchar(500),
        @Level varchar(100),
        @Message varchar(max),
        @ExceptionMessage varchar(max),
        @StackTrace varchar(max),
        @LoggerType varchar(100),
        @StartTime datetime,
        @EndTime datetime,
        @CostTime decimal(34,5),
        @ActionIdentifier varchar(500),
        @UserId varchar(100),
        @UserLoginName varchar(500),
        @Action varchar(500),
        @WebApiLogType varchar(100),
        @InContent varchar(max),
        @OutContent varchar(max)
    )
    AS
    BEGIN
        insert into
            LogDetail
        (
            [LogTime]
            ,[Category]
            ,[Level]
            ,[Message]
            ,[ExceptionMessage]
            ,[StackTrace]
            ,[LoggerType]
            ,[StartTime]
            ,[EndTime]
            ,[CostTime]
            ,[ActionIdentifier]
            ,[UserId]
            ,[UserLoginName]
            ,[Action]
            ,[WebApiLogType]
            ,[InContent]
            ,[OutContent]
        )
        values
        (
            @LogTime
            ,@Category
            ,@Level
            ,@Message
            ,@ExceptionMessage
            ,@StackTrace
            ,@LoggerType
            ,@StartTime
            ,@EndTime
            ,@CostTime
            ,@ActionIdentifier
            ,@UserId
            ,@UserLoginName
            ,@Action
            ,@WebApiLogType
            ,@InContent
            ,@OutContent
        )
    END
    GO
```
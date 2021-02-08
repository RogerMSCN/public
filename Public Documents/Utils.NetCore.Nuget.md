# Utils for .net core 5.0
## Summary
This is a collection of utils based on .net core 5.0. It will help you to manage a few normal fuctions:
- Write log.
  - Write log to log file.
  - Write log to a sql server database.
  - Write log to a dynamics 365 environment.
- Send request to dynamics 365 web api.

For these utils, you need to create user settings in default configuration file named "appsettings.json" with root section named "UserSettings".
## Write log
If you wanted to write log, please register dependency first.
#### Asp.Net Core
Register logger provider in ***Program.cs***.

Register necessary components in ***Startup.cs***.

In any models which include logger, create logger in constructor.

Then call functions to write log.
#### Application Core
Register dependencies in entry.
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
  - new_username
  - new_useryominame
  - new_webapilogtype
#### SQL Server implement
Create table.
Create store procedure.
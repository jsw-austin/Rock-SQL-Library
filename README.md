![Rock RMS SQL Library](https://raw.githubusercontent.com/SparkDevNetwork/Rock-SQL-Library/master/_assets/heading.jpg)

This is a public repo that contains SQL statements that are helpful in the day to day administration of Rock.

# Tips
To find a script you're interested in either select the directory that best matches what you're looking for, or use Github's search feature by typing `t` and a keyword.


# Contributing Guidelines
>SQL can be dangerous as it's outside of the castle wall. In Rock the C# models provide things like data integrity checks
and business logic validation. They also are much less prone to breaking changes. SQL in Rock should be used for 
administration and on rare cases reporting, but only after all other reporting options have been considered. **The goal of 
this repo is in no way an encouragement to add more dynamic SQL reports in your environment.** To do so would be to add more
risk to the day-to-day operations of your reporting strategy.

The goal of this repo is not to provide a list of every SQL script imaginable, but rather a highly curated
list of generally helpful and quality written SQL. If you have such a script feel free to send a PR. We 
will be selective of what we add, so please don't expect every PR to be accepted. Scripts not matching the
style guide rules below will be rejected.

## Directory Structure
Scripts should be placed into a directory with the Rock domain as the folder name. Not sure what domain
to use? Look for the directory name that the related block might be under. Some domains will have sub-directories
if we feel like it could have multiple scripts. For instance the `Interactions` directory has a sub-directory for
scripts around `Page Views`.

## What Not To Include
1. Scripts specific to a plugin. Exceptions will be made for _very_ popular plugins like the Room Manager.
2. Scripts that would be very rarely or insignificant. Sorry, we know this is subjective. 

## Styling Guide
~~~~
-- =====================================================================================================
-- Author:      Jon Edmiston
-- Create Date: 3/21/2020
-- Description: Displays page load times for a specified page. Also displays who loaded the page if 
--              known.
--              Note: Time is in seconds.
--
-- Change History:
--   3/22/2020 Jon Edmiston: Updated SQL to allow passing in Page Id vs having to know ComponentId.
-- =====================================================================================================

DECLARE @PageId int = 1247      -- Id of the page you want timings from. 
DECLARE @MaxRows int = 1000     -- The maximum number of rows to return.
DECLARE @DaysBack int = 1       -- The number of days back to look.

-------------------------------------------------------------------------------------------------------

SET @DaysBack = @DaysBack * -1
DECLARE @StartDate datetime = (SELECT DATEADD (day , @DaysBack , GETDATE() ) )

------------------------------

SELECT
    TOP (@MaxRows) 
    [InteractionTimeToServe] 
    , [InteractionDateTime]
    , ISNULL( p.[NickName], '') [First Name]
    , ISNULL( p.[LastName], '') [Last Name]
FROM
    [Interaction] i
    INNER JOIN [InteractionComponent] ic ON ic.[Id] = i.[InteractionComponentId]
    LEFT OUTER JOIN [PersonAlias] pa ON pa.[Id] = i.[PersonAliasId]
    LEFT OUTER JOIN [Person] p ON p.[Id] = pa.[PersonId]
WHERE
    ic.[EntityId] = @PageId 
    AND [InteractionTimeToServe] IS NOT NULL
    AND [InteractionDateTime] > @StartDate
ORDER BY [InteractionTimeToServe] DESC
~~~~

### Styling Elements
1. Be sure to add the intro comment section
   * Add a block Change History section for the next person
2. Declare variables for the person be able to adjust the query logic. The goal is that the individual should not need to tweak the SQL. They should just adjust the variables. Think of these as Pinocchio's strings (before becoming a boy)
3. The next section of variables is to adjust the variables to filter values. You may or may not need these. For example it's easier for the individual to say 100 days back then to have to calculate the date 100 days ago.
4. SQL should match the syntax of the Rock Developer Codex. 

## Tips and Tricks

### Add Caution Tape
 Be careful with queries that could ADD/UPDATE/DELETE data. You might consider a pattern like this for deletes.
~~~~
DECLARE @InteractionChannel int = 23

SELECT *
--DELETE
 FROM [Interaction] 
WHERE [InteractionComponentId] IN (SELECT [Id] FROM [InteractionComponent] WHERE [InteractionChannelId] = @InteractionChannel)

~~~~
_Note that if the individual runs this query it will initiall show them what would be deleted. They have to modify it a bit to actually perform the delete._

![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Only Backup SQL Principal Databases In A Mirror or AlwaysOn Configuration
**Post Date: March 21, 2016**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>The following logic can be placed into a Job by it's self. Here's what it does: 1. Creates a backup for all databases with an ONLINE state.
2. Creates a backup for all databases that are set as PRINCIPAL not MIRRORS. This will ensure that the databases will backup properly as Mirrored will error the Job. As a result this can be deployed on any SQL instance (mirrored or not), and will backup only databases that qualify.
3. Checks to see if certain features are enabled such as 'show advanced options', 'xp_cmdshell', and 'backup compression default'. They will be immediately enabled if they haven't been formerly set.
4. Adds a human-readable timestamp to the backup file name.
5. Checks to see if Databases are undergoing a current backup, log, or restore operation and if so; will ignore the database as not to cause an error or conflict.</p>      


## SQL-Logic
```SQL
use master;
set nocount on
 
declare @backup_all_databases   varchar(max)
declare @get_time       varchar(25)
declare @get_day        varchar(25)
declare @get_date       varchar(25)
declare @get_month      varchar(25)
declare @get_year       varchar(25)
declare @get_timestamp  varchar(255)
set     @get_day        = (select datename(dw, getdate()))
set     @get_date       = (select datename(dd, getdate()))
set     @get_month      = (select datename(mm, getdate()))
set     @get_year       = (select datename(yy, getdate()))
set     @get_time       = (select replace(replace(replace(replace(convert(char(20), getdate(), 22), '/', '-'), 'AM', 'am'), 'PM', 'pm'), ':', '-'))
set     @get_timestamp          = (select @get_time + ' ' + @get_month + ' ' + @get_day + ' ' + @get_date + ' ' + @get_year + ' Full Database Bu ')
set     @backup_all_databases   =   ''
select  @backup_all_databases   =   @backup_all_databases +
'
    if exists 
    (
    select 1 
    command from master.sys.dm_exec_requests where
    command in (''backup database'', ''backup log'', ''restore database'') 
    and db_name(database_id) = ''' + upper(name) + '''
    )
        begin
            print ''Database: [' + upper(name) + '] Has a backup or restore operation currently running.  Backup will be skipped.''
        end
        else
            backup database [' + upper(name) + '] to disk = ''E:\SQLBACKUPS\' + @get_timestamp + upper(name) + '.bak'' with format;
' + char(10)
from
    sys.databases sd join sys.database_mirroring sdm on sd.database_id = sdm.database_id
where
    name    not in ('tempdb')
    and     state_desc = 'online'
    and     sd.source_database_id   is null
    and     sdm.mirroring_role_desc is null
    or      sdm.mirroring_role_desc != 'mirror'
order by
    name asc
 
declare
    @sao    int = (select cast([value] as int) from master.sys.configurations where [name] = 'show advanced options')
,   @bcd    int = (select cast([value] as int) from master.sys.configurations where [name] = 'backup compression default')
,   @xpc    int = (select cast([value] as int) from master.sys.configurations where [name] = 'xp_cmdshell')
 
if  @sao = 0    begin exec  master..sp_configure 'show advanced options', 1         reconfigure end
if  @bcd = 0    begin exec  master..sp_configure 'backup compression default', 1    reconfigure end
if  @xpc = 0    begin exec  master..sp_configure 'xp_cmdshell', 1                   reconfigure end
exec    (@backup_all_databases)

```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

     
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

![Lee Songming](https://github.com/congmingshuju/git-resources/blob/master/images/0-clever-data-github.png "李聪明 数据")

# 将所有SQL Instance端口添加到防火墙
**发布-日期: 2016年11月4日 (评论)**

![Add Ports For SQL Instances](images/mikes_data_work_43340454_s.jpg?raw=true "SQL Firewall Rules")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL-Logic](#Logic)
- [中文-结果](#中文-结果)
- [English Results](#English-Results)
- [Build Quality](#Build-Quality)
- [Author](#Author)
- [License](#License) 


## 中文
以下是一些推断所有instance端口的SQL逻辑，同时创建了一个漂亮的netsh命令。
这样一来，你就可以把显示名称为“SQL Instance MyInstanceName”的端口添加到防火墙。


## English
Here’s some SQL logic that extrapolates all instances ports, and creates a nifty netsh command so you can add those ports to the firewall with a display name of "SQL Instance MyInstanceName".

---
## Logic
```SQL
use master;
set nocount on
 
declare @sql_instances      table ([rootkey] varchar(255), [value] varchar(255))
declare @firewall_commands  table ([int] int identity(1,1), [command] varchar(1000))
declare @run_commands       varchar(max) = ''
 
insert into @sql_instances 
exec master.dbo.xp_instance_regenumvalues 
    @rootkey    = N'HKEY_LOCAL_MACHINE'
,   @key        = N'SOFTWARE\\Microsoft\\Microsoft SQL Server\\Instance Names\\SQL';
 
declare db_cursor   cursor for select upper([rootkey]), upper([value]) from @sql_instances
declare @instance_name  varchar(255)
declare @instance_path  varchar(255)
open    db_cursor;
fetch next from db_cursor into @instance_name, @instance_path
    while @@fetch_status = 0  
        begin 
            declare @port   varchar(50)
            declare @key    varchar(255) = 'software\microsoft\microsoft sql server\' + @instance_path + '\mssqlserver\supersocketnetlib\tcp\ipall'
            exec master..xp_regread
            @rootkey        = 'hkey_local_machine'
            ,   @key        = @key
            ,   @value_name = 'tcpdynamicports'
            ,   @value      = @port output
            declare @add_firewall_rule  varchar(255)
            set @add_firewall_rule  = 'exec master..xp_cmdshell ''netsh advfirewall firewall add rule name="SQL Instance ' 
                            + upper(@instance_name) + '" dir=in action=allow protocol=tcp localport=' + isnull(convert(varchar(10), @port), 1433) + ''''
            insert into @firewall_commands
            select  (@add_firewall_rule)
            fetch next from db_cursor into @instance_name, @instance_path
        end;
    close db_cursor
deallocate db_cursor;
select  @run_commands = @run_commands + '' + command + ';' + char(10) from @firewall_commands
exec    (@run_commands)
```

![Adding Ports For ALL SQL Instances](images/image0012.png?raw=true "Adding Ports For All SQL Instances")

---

## 中文-结果
你会得到看起来如图上的结果，这个结果可以直接在服务器上运行，或者连接成变量，在作业上运行。只要SQL服务帐户在操作系统中拥有权限，端口将相应添加。使用上面的逻辑将其打包到一个在服务器上运行的变量@run_commands中，并添加防火墙规则。

## English-Results
Your results will look something like this which can be run from the server directly, or concatenated into variable, and run from a Job. As long as the SQL Service accounts have rights within the OS; the ports will be added accordingly. Using the logic above it is packaged into a variable @run_commands which is run on the server, and adds the firewall rules.


![Results](images/image0021.png?raw=true "Results")


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

---

## Build-Quality 
| [![Build status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg=true)](https://ci.appveyor.com/project/tygerbytes/resourcefitness) | [![Coveralls](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?branch=master)](https://coveralls.io/github/tygerbytes/ResourceFitness?branch=master) | [![nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](https://www.nuget.org/packages/TW.Resfit.Core/) |
|-|-|-|

>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](https://ci.appveyor.com/project/tygerbytes/resourcefitness/history)


## Author

- **李聪明 数据 Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明数据-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://github.com/congmingshuju/git-resources/blob/master/images/clever-data-gist-z5.png "李聪明 数据")

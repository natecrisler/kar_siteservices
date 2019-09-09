# Script Purpose
 A ```.path ``` file is created to be used as a wrapper for the powershell ```.ps1``` script that is to be run.

 ## Path File
 ``` driveinfo.path ```

### Contents

```
%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe -command "& {set-executionpolicy Remotesigned; '$SPLUNK_HOME\etc\apps\kar_siteservices\bin\driveInfo.ps1'}"
```



 ## Powershell Script

 ``` driveInfo.ps1 ```
# Script Purpose

The purpose of this script is to look at each drive and determine what is the cause of the disk usage and what is taking up the most space. This will assist the Field Administrators in the troubleshooting process to cleaning up disk space.

### Contents


```
##Mention the path to search the files
#$path = "\\p-long-file-01\f$\User_Data"
#$path = "\\olme-l-83333sn\c$\users\coye.brabham

#Location to save the files to
$FilePath = "C:\Users\todd.beedy\Dropbox\FAA IT\Adesa\Site_Services\csv_reports\"

#File Server to run against
$FileServer = "p-tuls-file-01"

#This is local Path to run through.
$path = "F:\"

##Find out the files greater than equal to below mentioned size
$size = 10MB

##Limit the number of rows
$limit = 100000

##Find out the specific extension file
#$Extension = "*.jpg"
#$Extension = "*.*"

$String = @"
    `$AllFiles = Get-ChildItem -Path $path -Attributes `"!Directory`" -recurse
    `$FileResults  = @()
    `$FileExt = @{}
    `$FileFolder = @{}
    `$Outputtext = @()

    `$Outputtext += `"Found `$(`$AllFiles.count) Total Files`"

    foreach(`$file in `$AllFiles){

        if(`$file.length -gt $size){
            if(`$FileResults.count -gt $limit){
                break
                }
            else{

                #Add to Hashtables :)
                `$FileExt["`$($`File.Extension)"]++
                `$FileFolder["`$(`$File.Directory.fullname)"]++
                `$FileResults  += `$File | select Name, @{Name=`"SizeInMB`";Expression={`$`_.Length / 1MB}},@{Name=`"Path`";Expression={`$`_.directory}}
                }
            }
        }
    `$Outputtext += `"Found `$(`$FileResults.count) Files Over $($Size/1mb) MB`"

    `$FileExt,`$FileResults,`$FileFolder,`$OutputText

"@

##script to find out the files based on the above input
$SB = [scriptblock]::Create($String)
$largeSizefiles = Invoke-Command -ScriptBlock $SB -ComputerName $FileServer

#Export All Files over size
$largeSizefiles[1] | Export-Csv "$Filepath\$FileServer._all.csv" -NoTypeInformation
#Export File Extensions in violation of size
$largeSizefiles[0].GetEnumerator() | select name,value | sort value -Descending | Export-Csv "$Filepath\$FileServer._Ext.csv" -Force -NoTypeInformation
#Folder Item Count
$largeSizefiles[2].GetEnumerator() | select name,value | sort value -Descending | Export-Csv "$Filepath\$FileServer._Folder.csv" -Force -NoTypeInformation

#Output Stats :).
$largeSizefiles[3]

#Export

#>


<#
    $AllFiles = Get-ChildItem -Path $path -Attributes "!Directory" -recurse
    $FileResults  = @()
    $FileExt = @{}
    $FileFolder = @{}

    foreach($file in $AllFiles){

        if($file.length -gt $size){
            if($FileResults.count -gt $limit){
                break
                }
            else{

                #Add to Hashtables :)
                $FileExt["$File.Extension"]++
                $FileFolder["$File.Directory"]++
                $FileResults  += $File | select Name, @{Name="SizeInMB";Expression={$_.Length / 1MB}},@{Name="Path";Expression={$_.directory}}
                }
            }
        }
    $FileExt,$FileFolder,$FileResults

#>
```

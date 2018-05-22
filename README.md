### NOTE: This is to rescue the psasync project from it's original location at [CodePlex](https://archive.codeplex.com/?p=psasync), now archived. It was created for PowerShell v2, but probably still works.

psasync is a PowerShell module that makes executing multi-threaded PowerShell easier. It provides significant performance benefits over the standard PoSH job system.

The psasync module contains the following functions:  
### Get-RunspacePool

This should have, perhaps, been named Create-RunspacePool. As its name suggests, it returns a pool of runspaces that can be used (and reused).

### Invoke-Async

This is a much improved version of the function introduced in the initial post. The big improvement was the addition of the AsyncPipeline class definition, which allows the creation of a simple object to keep track of both the pipeline and the AsyncResult handle which is returned by BeginInvoke(). This allows the process of looking at statuses of running processes and consuming results to be much simpler. The function also allows passing an array of parameters for script blocks with multiple arguments.

### Receive-AsyncResults

This function wraps the code for pulling results (or errors, as the case may be) off the pipelines in the runspace pool utilizing the AsyncPipeline objects output from Invoke-Async.
Receive-AsyncStatus

A handy function for working with runspaces from the shell, Receive-AsyncStatus simply returns information about the status of the operations running in the pipelines you have invoked. Since Receive-AsyncResults is synchronous, this allows you to continue to work until your last process completes or selectively use Receive-AsyncResults on those that have completed.
Example Code

To demonstrate the use of the module, consider the following scenario. You have a series of Excel documents that you need to load into an SQL Server database. As before, set up the script block that will execute the real work.

```PowerShell
Import-Module psasync

$AsyncPipelines = @()

$ScriptBlock = `
{
    Param($File)
    
    . <your_path>\Import-ExcelToSQLServer.ps1
    
    Import-ExcelToSQLServer -ServerName 'localhost' -DatabaseName 'SQLSaturday' -SheetName "SQLSaturday_1" `
        -TableName $($File.BaseName) -FilePath $($File.FullName)
}

# Create a pool of 3 runspaces
$pool = Get-RunspacePool 3

$files = Get-ChildItem <path-to-files> 

foreach($file in $files)
{
	 $AsyncPipelines += Invoke-Async -RunspacePool $pool -ScriptBlock $ScriptBlock -Parameters $file
}

Receive-AsyncStatus -Pipelines $AsyncPipelines

Receive-AsyncResults -Pipelines $AsyncPipelines -ShowProgress
```

You’ll notice there is nothing particularly complex here in the code. But, all the warnings from the runspace post apply. Multi-threading is awesome and powerful, but use it with care. 

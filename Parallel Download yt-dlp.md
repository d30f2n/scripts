# How to Batch Download in Parallel with yt-dlp using PowerShell

## Prerequisites
ThreadJob Module for PowerShell
To install, run `Install-Module -Scope CurrentUser -Name ThreadJob` in PowerShell.

## Usage
- `yt-dlp-parallel path\to\listOfUrls.txt` will batch download URLs in parallel listed in a text document

## Instructions
1. Create a PowerShell profile if you haven't before. You will place the script in your profile so you can run it at any time.
   ```
   if (!(Test-Path -Path $PROFILE)) {
       New-Item -ItemType File -Path $PROFILE -Force
   }
   ```
2. Open your profile in notepad by typing in `notepad $profile`.
3. Append the script below to your profile.
4. Make the changes using the guidelines in the **Editing the script** section.
5. Save and close notepad.
6. Close and reopen PowerShell.

## Editing the script
- This script will work as is. If you want to use my settings, you do not have to change anything.
- Change the function name to whatever you want. This is what you will type in to run yt-dlp. I have mine as `yt-dlp-parallel`.
- Change maxThreads to the amount of concurrent downloads you want. Some sites might throttle you to a certain amount of connections so you shouldn't make this number too high.
- I have my default output file template as just the title. If you want something else, change `'%(title)s.%(ext)s'` to your liking. Refer to [the output template](https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#output-template) for more information.
- When parallel downloading, it will show the yt-dlp output from all jobs in the current console window. This is why I disabled the progress bar while in parallel. If you do not want any logging messages showing, you can remove this section:
  ```
      foreach($id in Get-Job)
    {
        $numIds++
    }
    $idArray = 1..$numIds
    Receive-Job -Id $idArray -Wait
  ```
  If you remove this, the tasks will continue to run in the background without any streaming output. You can run `Get-Job` to see which yt-dlp jobs have finished. Then `Receive-Job -Id <idnumber>` to get the yt-dlp messages from that job.

## Script
```
function yt-dlp-parallel([string]$inputFile)
{
    $maxThreads = 3
    Write-Output "Running yt-dlp in parallel using $maxThreads threads"
    foreach($line in Get-Content $inputFile)
    {
        Start-ThreadJob -ScriptBlock {yt-dlp -o '%(title)s.%(ext)s' --no-progress $Using:line} -ThrottleLimit $maxThreads
    }
    foreach($id in Get-Job)
    {
        $numIds++
    }
    $idArray = 1..$numIds
    Receive-Job -Id $idArray -Wait
}
```

## Resources
- yt-dlp https://github.com/yt-dlp/yt-dlp
- PowerShell Profiles https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles
- ThreadJob https://learn.microsoft.com/en-us/powershell/module/threadjob/
- Start-ThreadJob https://learn.microsoft.com/en-us/powershell/module/threadjob/start-threadjob
- Get-Job https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-job
- Receive-Job https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/receive-job

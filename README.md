# Remote-Sync

While migrating windows files from one PC to another missed an rsync tool on windows. With some ideas from forums, got some usable poweshell code, copying files and preserving timestamps:

```powershell
function rsync  {
  param ( $source,
          $target 
        )

  $WorkDir=Get-Location
  Write-Output "PATH PWD:   $WorkDir"
  Write-Host "SOURCE: $source"
  Write-Host "TARGET: $target"

  $sourceFiles = Get-ChildItem -Path $source -Recurse
  $targetFiles = Get-ChildItem -Path $target -Recurse

  Write-Host "SOURCEFILES $sourceFiles"
  Write-Host "TARGETFILES: $targetFiles"


  if ($debug -eq $true) {
    Write-Output "Source=$source, Target=$target"
    Write-Output "sourcefiles = $sourceFiles TargetFiles = $targetFiles"
  }
  <#
  1=1 way sync, 2=2 way sync, 3=1 way purging target files not equal with source files
  #>
  $syncMode = 3

  if ($targetFiles -eq $null)
  {
      $targetFiles=""
  }

  if ($sourceFiles -eq $null){
    Write-Host "o" -NoNewline
  } else
  {
    $diff_command="Compare-Object -ReferenceObject $sourceFiles -DifferenceObject $targetFiles"

    $diff = Compare-Object -ReferenceObject $sourceFiles -DifferenceObject $targetFiles

    foreach ($f in $diff) {
      if ($f.SideIndicator -eq "<=") {
        $fullSourceObject = $f.InputObject.FullName
        $fullTargetObject = $f.InputObject.FullName.Replace($source,$target)

        Write-Host "." -NoNewline

        Copy-Item -Path $fullSourceObject -Destination $fullTargetObject
        $origLastWriteTime = ( Get-Item $fullSourceObject ).LastWriteTime
        $origCreationTime = ( Get-Item $fullSourceObject ).CreationTime
        (Get-Item $fullTargetObject).LastWriteTime = $origLastWriteTime
        (Get-Item $fullTargetObject).CreationTime = $origCreationTime
      }


      if ($f.SideIndicator -eq "=>" -and $syncMode -eq 2) {
        $fullSourceObject = $f.InputObject.FullName
        $fullTargetObject = $f.InputObject.FullName.Replace($target,$source)

        Write-Host "." -NoNewline

        Copy-Item -Path $fullSourceObject -Destination $fullTargetObject
        $origLastWriteTime = ( Get-Item $fullSourceObject ).LastWriteTime
        $origCreationTime = ( Get-Item $fullSourceObject ).CreationTime
        (Get-Item $fullTargetObject).LastWriteTime = $origLastWriteTime
        (Get-Item $fullTargetObject).CreationTime = $origCreationTime
      }

      if ($f.SideIndicator -eq "=>" -and $syncMode -eq 3 -and $targetFiles -ne "") {
        $fullSourceObject = $f.InputObject.FullName
        $fullTargetObject = $f.InputObject.FullName.Replace($target,$source)

        Write-Host "x" -NoNewline

        Remove-Item  $fullSourceObject 
      }

    }
  }
}
```

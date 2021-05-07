# PowerShell code snips

File selection function

    Function Get-FileName($initialDirectory)
    {
        [System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms") | Out-Null

        $OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
        $OpenFileDialog.initialDirectory = $initialDirectory
        $OpenFileDialog.filter = "CSV (*.csv)| *.csv"
        $OpenFileDialog.ShowDialog() | Out-Null
        $OpenFileDialog.filename
    }

Load a CSV file

    function LoadFile($filename){
      Write-Output "Loading File: $filename"
      $fileContent = Import-Csv -Path $filename
      write-output "Loading File Complete, $($fileContent.Count) entries loaded"
      return $fileContent
    }

Iterate over the CSV filename

    $list = LoadFile(Get-FileName())

    foreach ($item in $list)
    {
    //Do Something
    }

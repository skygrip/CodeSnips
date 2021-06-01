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

    $filename = Get-FileName
    function Load-CSVFile($filename){
      Write-Output "Loading File: $filename"
      $fileContent = Import-Csv -Path $filename
      write-output "Loading File Complete, $($fileContent.Count) entries loaded"
      return $fileContent
    }

Iterate over the CSV filename

    $list = Load-CSVFile(Get-FileName())

    foreach ($item in $list)
    {
    //Do Something
    }


Do Mass DNS Lookups

    # CSV File must have a column called Query with a list of queries
    $List = Load-CSVFile(Get-FileName())
    $List.Query | resolve-dnsname | where {$_.Section -eq "Answer"} | Select Name, IPAddress | Sort Name
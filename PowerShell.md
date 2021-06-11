# PowerShell code snips

File selection function

    Function Get-FileName($initialDirectory)
    {
        [System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms") | Out-Null

        $OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
        $OpenFileDialog.initialDirectory = $initialDirectory
        # $OpenFileDialog.filter = "CSV (*.csv)| *.csv"
        $OpenFileDialog.ShowDialog() | Out-Null
        $OpenFileDialog.filename
    }

Load a CSV file

    $filename = Get-FileName

    function Load-CSVFile($filename){
      Write-Host "Loading File: $filename"
      $fileContent = Import-Csv -Path $filename
      Write-Host "Loading File Complete, $($fileContent.Count) entries loaded"
      return $fileContent
    }

Iterate over the CSV filename

    # Load the file
    $filename = Get-FileName()
    $list = Load-CSVFile()

    # Add a Results column
    $list = $list | Select *,"Result"

    foreach ($item in $list) {
        # reminder, $item is a reference so changes to it 
        # are reflectedin $list, just dont change the structure

        try {
            $item.Result = Some Command
        }
        catch{
            Write-Warning -Message "error"
            $item.Result = "error"
        }
    }

    # Save the results
    $list | Export-Csv -Path $filename -NoTypeInformation

Do Mass DNS Lookups

    # CSV File must have a column called Query with a list of queries
    $List = Load-CSVFile(Get-FileName())
    $List.Query | resolve-dnsname | where {$_.Section -eq "Answer"} | Select Name, IPAddress | Sort Name
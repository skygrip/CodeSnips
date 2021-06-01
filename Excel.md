# Excel Code Snips

Check if an item is in a list

    # Exact Match
    =COUNTIF(SheetList!$A:$A,A2)>0

    # Begins with A2
    =COUNTIF(SheetList!$A:$A,A2 & "*")>0

Do a lookup on an item in excel

    =VLOOKUP(A2,SheetLookup!$A:$A,2,FALSE)


Do a Lookup on a Subnet in excel

    # Import the VBA https://github.com/andreafortuna-org/VBAIPFunctions
    # Warning, dont specify too large a range as it may crash excel
    =IpSubnetVLookup(A2,networks!$A$1:$B$99,2)

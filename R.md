# R Code Snips

Work out the difference between two dates

    library(lubridate)
    a=as.Date("2022-03-12")
    b=as.Date("2022-07-15")
    # As Days
    interval(a,b) %/% days(1)
    # As Months
    interval(a,b) %/% months(1)

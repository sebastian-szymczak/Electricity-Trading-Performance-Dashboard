# Statistics for Contracts with Delivery of Electricity @ Organised Trading Facility / Towarowa Giełda Energii S.A.

### Dashboard link: https://app.powerbi.com/groups/me/reports/e195a17e-465f-4090-bcdc-139723f3882c/ReportSection

## Problem Statement

The report is used to determine price trends of particular futures contracts (BASE and PEAK5) for electricity for periods of not less than one month.
Calculated parameters are very important in terms of the market, because on their basis the trading companies buy electrical energy for households and enterprises, and then, based on it, they create prices for these entities.

### Steps followed

- Step 1: An .xlsm file containing data from the stock market was loaded into Power BI Desktop.
- Step 2: In power query editor, "RTTE" sheet was chosen as the source of dataset.

  The following column was added in order to create data model:

          int = FORMAT('calendar'[date], "YYYYMMDD")  

  Also the index column was added.
  
- Step 3: An .xlsm file containing information about which days the stock market was opened, was loaded into Power BI Desktop.

  The following column was added in order to create data model:

          int = FORMAT('calendar'[date], "YYYYMMDD")  

- Step 4: Using DAX expressions, the calendar table was created and marked as date table:

          calendar = CALENDAR(DATE(2013,1,1), MAXX(rtee, rtee[date]))

  The following columns were added in order to create data model and slicers:

          int = FORMAT('calendar'[date], "YYYYMMDD")

          year = YEAR(calendar[date])

          month = FORMAT('calendar'[date],"mmmm")
  
          no_of_month = MONTH(calendar[date])
  
- Step 5: The creation of relationships between calendar[int] and rtee[int], as well as calendar[int] and working_days[int], in order to constructing data model:

![model](https://github.com/user-attachments/assets/03033839-db27-416b-a338-143bfd61b7ab)

- Step 6: Visual filters (slicers) were added for each type of Contract and every month and year.
- Step 7: Using DAX expressions, new measures were created to calculate the following parameters:
  
  (a) Weighted Average Price

          wavg_price = DIVIDE(SUMX(rtee, rtee[price]*rtee[volume]), SUMX(rtee, rtee[volume]))

  (b) Daily Minimum Price

          daily_min = MINX(rtee, rtee[price])
  
  (c) Daily Maximum Price

          daily_max = MAXX(rtee, rtee[price])

  (d) Last Transaction Price

          last_tr_price = CALCULATE(LASTNONBLANKVALUE(rtee[date], AVERAGE(rtee[price])), ALLSELECTED(calendar[date]))

The main line chart was used to represent the values of all parameters above as a function of chosen period of time.

![main](https://github.com/user-attachments/assets/25cf7194-9607-4429-8d10-ef7bc4066956)
  
- Step 8: The column rtee[volume] was used to calculate the following parameters:

  (a) Total Trades - as the count of all values

  (b) Volume - as the sum of all values

- Step 9: Using DAX expressions, new measures were created to calculate the following parameters:
  
  (a) Minimum Price

          price_min = 
          var x = CALCULATE(MINX(rtee, rtee[price]), ALLSELECTED('calendar'[date]))
          return
          if (MIN(rtee[price]) = x, x, BLANK())
  
  (b) Maximum Price

          price_max = 
          var x = CALCULATE(MAXX(rtee, rtee[price]), ALLSELECTED('calendar'[date]))
          return
          if (MAX(rtee[price]) = x, x, BLANK())
   
  (c) DKR (Daily Clearing Price) - the parameter determined as the average price of the last 10 transactions (or less if there were fewer transactions) on a given   day.

          dkr = LASTNONBLANKVALUE('calendar'[date],
                    CALCULATE(
                        AVERAGEX(TOPN(10, rtee, rtee[index], desc, rtee[price], ASC), rtee[price])
                    )
                )
  
  (e) YTD (Year to Date)

          ytd = CALCULATE([wavg_price], 'calendar'[date] <= MAX('calendar'[date]), DATESYTD('calendar'[date]))
 
  (f) Average Daily Volume

          avg_volume = DIVIDE(SUM(rtee[volume]), SUM(working_days[working]))
         
A set of card visuals was used to represent the values of all indicators above, as well as "Weighted Average Price" (step 7a), "Total Trades" (step 8a) and "Volume" (step 8b) parameters.

![cards](https://github.com/user-attachments/assets/de509c01-6fa0-46b2-97b1-cf55f4d5d581)
        
- Step 10: Using DAX expressions, new measure was created to calculate the difference between weighted average and YTD prices.

          diff_ytd = IF ([wavg_price]=BLANK(), BLANK(), [wavg_price]-[ytd])

A clustered column chart, representing the difference as a function of chosen period of time, was added.

![diff_ytd](https://github.com/user-attachments/assets/d8baaf09-2c3f-4516-aaa7-85375530a133)

- Step 11: A clustered column chart, representing the volume changes as a function of chosen period of time, was added.

![vol](https://github.com/user-attachments/assets/9cb46539-b765-4dcf-82dd-344886f891d0)

- Step 12: A standalone visual date filter (slicer) for last two weeks range was added in the bottom right corner. It allows the user to view both date, time and price of each concluded transaction within a selected period, on the line chart below. 

![tracker](https://github.com/user-attachments/assets/379da0aa-5d0c-43bd-afe9-f3c75393a853)

To work this properly, the dataset should be kept up to date.

- Step 13: The report was published to Power BI Service.

![publish](https://github.com/user-attachments/assets/5a19c892-7b86-4961-89bf-b741ad5f1353)

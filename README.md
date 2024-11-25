# PowerBI-DAX-Measures

This repository contains DAX measures used in Power BI projects. Each measure includes an explanation of its purpose, logic, and usage context.

## Measures

### 1. TotalBeneficiaries

- **Purpose**: Calculates the total number of beneficiaries grouped by Sub-region, Intervention Type, WASH Domain, and Activity. It also includes custom logic to handle maximum values for specific domains and intervention types.
- **Key Logic**:
  1. Groups data by Sub-region, Intervention Type, WASH Domain, Activity, and Reporting Date.
  2. Calculates the maximum activity sum for each month.
  3. Sums all maximum values and adds a filtered maximum for specific conditions.
- **Usage**: This measure is used in a card visual to display a summary of beneficiaries, with special emphasis on IDP and WASH domain activities.

- **DAX Code**:
  ```DAX
  TotalBeneficiaries = 
  VAR GroupedTable = 
      SUMMARIZE(
          'WASH 5W Dataset',
          'WASH 5W Dataset'[Sub-region],
          'WASH 5W Dataset'[Intervention / Emergency Type],
          'WASH 5W Dataset'[WASH Domain],
          'WASH 5W Dataset'[Activity],
          'WASH 5W Dataset'[Reporting Date],
          "MonthlySum", SUM('WASH 5W Dataset'[Total Beneficiaries])
      )
  
  VAR MaxPerMonth =
      ADDCOLUMNS(
          SUMMARIZE(
              GroupedTable,
              'WASH 5W Dataset'[Sub-region],
              'WASH 5W Dataset'[Intervention / Emergency Type],
              'WASH 5W Dataset'[WASH Domain],
              'WASH 5W Dataset'[Reporting Date]
          ),
          "MaxActivitySum", MAXX(
              FILTER(
                  GroupedTable,
                  'WASH 5W Dataset'[Sub-region] = EARLIER('WASH 5W Dataset'[Sub-region]) &&
                  'WASH 5W Dataset'[Intervention / Emergency Type] = EARLIER('WASH 5W Dataset'[Intervention / Emergency Type]) &&
                  'WASH 5W Dataset'[WASH Domain] = EARLIER('WASH 5W Dataset'[WASH Domain]) &&
                  'WASH 5W Dataset'[Reporting Date] = EARLIER('WASH 5W Dataset'[Reporting Date])
              ),
              [MonthlySum]
          )
      )
  
  VAR SumOfMaxActivity = 
      SUMX(MaxPerMonth, [MaxActivitySum])
  
  VAR FilteredMax = 
      MAXX(
          FILTER(
              MaxPerMonth,
              'WASH 5W Dataset'[Intervention / Emergency Type] = "IDP" &&
              ('WASH 5W Dataset'[WASH Domain] = "Water" || 'WASH 5W Dataset'[WASH Domain] = "Hygiene")
          ),
          [MaxActivitySum]
      )
  
  RETURN
      SumOfMaxActivity + FilteredMax
  ``

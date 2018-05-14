# SHINE
```sql

/* Adding direct fields*/

SELECT    t_all.[Source], 
          t_all.[Calendar Day of Year Number], 
          t_all.[Outlet Key], 
          t_all.[Teradata Material Key], 
          t_all.[Volume], 
          t_all.[CMA], 
          t_all.[BI NSI],         
          t_1.[Material Description], 
          t_1.[Size], 
          t_1.[Conversion RAW to SPC], 
          t_1.[Local COGS diff], 
          t_1.[Buyback], 
          t_2.[Outlet], 
          t_2.[Credit to Outlet Key(CTO)], 
          t_2.[Agent], 
          t_2.[Legacy Key Account Key], 
          t_2.[Legacy Key Account], 
          t_2.[Customer Per Rate Schedule], 
          t_2.[Cust_Type], 
          t_2.[Region] 
INTO 0_YTD_Transactions_1stUpdate IN 'C:\Users\B80883\Downloads\Databases\005_Data_Dump.accdb'
FROM ((0_YTD_Transactions AS t_all 
     LEFT JOIN a_Matterial_Lookup AS t_1 
            ON t_all.[Teradata Material Key] = t_1.[Material #]) 
     LEFT JOIN 0_All_Customers_and_Agents AS t_2 
            ON t_all.[Outlet Key] = t_2.[Outlet Key]) 
     LEFT JOIN [#_Cost_and_Buyback] AS t_3 
            ON t_all.[Teradata Material Key] = t_3.[Material];

/* Category 1 - MMBU, Freestyle and Allied*/

SELECT t_update.[Outlet Key], 
       t_update.[Teradata Material Key],
       IIf([t_1].[Grouping]="MMBU","MMBU",IIf([t_1].[Grouping]="Freestyle","Freestyle",IIf([t_1].[Grouping]="Allied","Allied","Others"))) AS  [Category]
INTO Temp_Category1 IN 'C:\Users\B80883\Downloads\Databases\005_Data_Dump.accdb'
FROM Temp_YTD_Transactions_1stUpdate AS t_update
LEFT JOIN a_Matterial_Lookup AS t_1 
ON t_update.[Teradata Material Key]=[t_1].[Material #]
GROUP BY t_update.[Outlet Key], 
         t_update.[Teradata Material Key],
         IIf([t_1].[Grouping]="MMBU","MMBU",IIf([t_1].[Grouping]="Freestyle","Freestyle",IIf([t_1].[Grouping]="Allied","Allied","Others")));
         

/* Category 2 - Special Customers, Double Counting Identifier, CCL Agent*/

SELECT    t_update.[Calendar Day of Year Number], 
          t_update.[Outlet Key], 
          t_update.[Teradata Material Key], 
          IIF (
          t_update.[Legacy Key Account Key]=t_2.[Legacy Key Account Key],"CCL CWD Customers",
                    IIF((t_1.[Distributer Identifier]="Yes" 
                    AND t_update.[Legacy Key Account Key]<>t_2.[Legacy Key Account Key]),"Distributors who report volume",
                           IIF((t_1.[Cust Group - per CCL] = "CCL-Agent" 
                                    AND t_1.[Distributer Identifier]<>"Yes" 
                                    AND t_update.[Legacy Key Account Key]<>t_1.[Legacy Key Account Key]),
                                                  "CCL-Agent", "CCC"))
          ) AS Category 

INTO Temp_Category2 IN 'C:\Users\B80883\Downloads\Databases\005_Data_Dump.accdb'
FROM Temp_YTD_Transactions_1stUpdate AS t_update, [#_Special_Customers] as t_2
LEFT JOIN a_Customer_Lookup as t_1
ON (t_update.[Legacy Key Account Key]=t_1.[Legacy Key Account Key])
WHERE (t_update.[Category] not in ("MMBU","Freestyle","Allied"))
GROUP BY  t_update.[Calendar Day of Year Number], 
          t_update.[Outlet Key], 
          t_update.[Teradata Material Key], 
          IIF (
          t_update.[Legacy Key Account Key]=t_2.[Legacy Key Account Key],"CCL CWD Customers",
                    IIF((t_1.[Distributer Identifier]="Yes" 
                    AND t_update.[Legacy Key Account Key]<>t_2.[Legacy Key Account Key]),"Distributors who report volume",
                           IIF((t_1.[Cust Group - per CCL] = "CCL-Agent" 
                                    AND t_1.[Distributer Identifier]<>"Yes" 
                                    AND t_update.[Legacy Key Account Key]<>t_1.[Legacy Key Account Key]),
                                                  "CCL-Agent", "CCC"))
          );

```

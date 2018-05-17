# SHINE
```sql

/* 001 Adding direct fields*/

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

/* 002 Temp Category 1 - MMBU, Freestyle and Allied*/

SELECT    t_update.[Calendar Day of Year Number], 
          t_update.[Outlet Key], 
          t_update.[Teradata Material Key], 
          t_update.[Legacy Key Account Key], 
          t_update.[Credit to Outlet Key(CTO)], 
          IIf([t_1].[Grouping]="MMBU","MMBU",
                    IIf([t_1].[Grouping]="Freestyle","Freestyle",
                              IIf([t_1].[Grouping]="Allied","Allied","Others"))) AS Category
INTO Temp_Category1 IN 'C:\Users\B80883\Downloads\Databases\005_Data_Dump.accdb'
FROM Temp_YTD_Transactions_1stUpdate AS t_update 
          LEFT JOIN a_Matterial_Lookup AS t_1 ON t_update.[Teradata Material Key] = t_1.[Material #]
GROUP BY  t_update.[Calendar Day of Year Number], 
          t_update.[Outlet Key], 
          t_update.[Teradata Material Key], 
          t_update.[Legacy Key Account Key], 
          t_update.[Credit to Outlet Key(CTO)], 
           IIf([t_1].[Grouping]="MMBU","MMBU",
                    IIf([t_1].[Grouping]="Freestyle","Freestyle",
                              IIf([t_1].[Grouping]="Allied","Allied","Others")));

         

/* 003 Temp Category all - hierarchy: MMBU, Freestyle and Allied, Special Customers, Double Counting Identifier, CCL Agent, CCRC, CCL*/

SELECT    t_temp.[Calendar Day of Year Number], 
          t_temp.[Outlet Key], 
          t_temp.[Teradata Material Key], 
          IIf(t_temp.[Category]<>"Others",t_temp.[Category],
                    IIf(t_temp.[Legacy Key Account Key] = t_sp.[Legacy Key Account Key],"CCL CWD Customers",
                              IIf(t_ag.[Double Count Indicator]="YES","Distributors who report volume",
                                        IIf(t_cust.[Cust_Type]="CCL-Agent","CCL-Agent",
                                                  IIF(t_cust.[Cust_Type]="CCRC","CCRC",  
                                                            IIF(t_cust.[Cust_Type]="CCL","CCL","CCC")))))) AS Category2 
INTO Temp_Category2 IN 'C:\Users\B80883\Downloads\Databases\005_Data_Dump.accdb'
FROM ((Temp_Category1 AS t_temp 
          LEFT JOIN [#_Special_Customers] AS t_sp ON t_temp.[Legacy Key Account Key] = t_sp.[Legacy Key Account Key]) 
          LEFT JOIN a_Customer_Lookup AS t_cust ON t_temp.[Legacy Key Account Key] = t_cust.[Legacy Key Account Key]) 
          LEFT JOIN a_Agent_Lookup AS t_ag ON t_temp.[Credit to Outlet Key(CTO)] = t_ag.[Credit to Outlet Key(CTO)]
GROUP BY  t_temp.[Calendar Day of Year Number], 
          t_temp.[Outlet Key], 
          t_temp.[Teradata Material Key], 
          IIf(t_temp.[Category]<>"Others",t_temp.[Category],
                    IIf(t_temp.[Legacy Key Account Key] = t_sp.[Legacy Key Account Key],"CCL CWD Customers",
                              IIf(t_ag.[Double Count Indicator]="YES","Distributors who report volume",
                                        IIf(t_cust.[Cust_Type]="CCL-Agent","CCL-Agent",
                                                  IIF(t_cust.[Cust_Type]="CCRC","CCRC",  
                                                            IIF(t_cust.[Cust_Type]="CCL","CCL","CCC"))))));


/* 004 Temp_Product_Grouping_Adding (General Groupings)*/

SELECT    t_update.[Outlet Key], 
          t_update.[Teradata Material Key], 
          t_update.[Calendar Day of Year Number], 
          t_update.[Customer Per Rate Schedule], 
          t_mat.[Grouping] INTO Temp_Product_Grouping1 IN 'C:\Users\B80883\Downloads\Databases\005_Data_Dump.accdb'
FROM a_Matterial_Lookup AS t_mat, Temp_YTD_Transactions_1stUpdate AS t_update
WHERE ((t_update.[Teradata Material Key] = t_mat.[Material #] and (t_mat.[Customer Specific]) is null));

/* 005 Temp_Product_Grouping_Updating (Specific Groupings))*/

SELECT    t_group.[Outlet Key], 
          t_group.[Teradata Material Key], 
          t_group.[Calendar Day of Year Number], 
          IIF((t_group.[Customer Per Rate Schedule] = t_mat.[Customer Specific ]) 
                    AND (t_group.[Teradata Material Key] = t_mat.[Material #]), t_mat.[Grouping], t_group.[Grouping]) AS [Product Grouping] 
INTO Temp_Product_Grouping2 IN 'C:\Users\B80883\Downloads\Databases\005_Data_Dump.accdb'
FROM Temp_Product_Grouping1 AS t_group 
          LEFT JOIN a_Matterial_Lookup AS t_mat 
          ON t_group.[Customer Per Rate Schedule] = t_mat.[Customer Specific] AND t_group.[Teradata Material Key] = t_mat.[Material #]
GROUP BY  t_group.[Outlet Key], 
          t_group.[Teradata Material Key], 
          t_group.[Calendar Day of Year Number], 
          IIF((t_group.[Customer Per Rate Schedule] = t_mat.[Customer Specific]) 
                    AND (t_group.[Teradata Material Key] = t_mat.[Material #]), t_mat.[Grouping], t_group.[Grouping]);

/* 006 Fetch customer price from Shashi's list*/
SELECT    t_update.[Calendar Day of Year Number], 
          t_update.[Outlet Key], 
          t_update.[Teradata Material Key], 
          t_rate.[Price to Customer] 
INTO Temp_NSI IN 'C:\Users\B80883\Downloads\Databases\006_Data_Dump.accdb'
FROM Temp_YTD_Transactions_2ndUpdate as t_update 
     LEFT JOIN [#_Rate_Schedule]  as t_rate
        ON  (t_update.[Calendar Day of Year Number]<(format(t_rate.[End Date Price to Customer],"00000")-43100)) 
        AND (t_update.[Calendar Day of Year Number]>(format(t_rate.[Start Date Price to Customer],"00000")-43100)) 
        AND (t_update.[Product Grouping] = t_rate.[Product Grouping]) 
        AND (t_update.[Customer Per Rate Schedule] = t_rate.Customer)
        AND (t_update.[Region] = t_rate.[Region])
GROUP BY t_update.[Calendar Day of Year Number], t_update.[Outlet Key], t_update.[Teradata Material Key], t_rate.[Price to Customer];


```

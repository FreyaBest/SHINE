# SHINE
```sql


SELECT		t_all.[Source],
          t_all.[Calendar Day of Year Number],
          t_all.[Outlet Key],
          t_all.[Teradata Material Key],
          t_all.[Volume],
          t_all.[CMA],
          t_all.[BI],
          t_all.[NSI]

INTO 		0_YTD_Transactions_1stUpdate
FROM		0_YTD_Transactions as t_all
RIGHT JOIN  	[a_Matterial_Lookup] AS t_1, [0_All_Customers_and_Agents] as t_2, [#_Cost_and_Buyback] as t_3 
ON 		((t_all.[Outlet Key]) = t_2.[Outlet Key])
      ((t_all.[Teradata Material Key]) = t_1.[Material #])
      ((t_all.[Teradata Material Key]) = t_3.[Material])
      

```

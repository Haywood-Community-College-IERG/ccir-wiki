- [Python Cheat-sheet](#python-cheat-sheet)
    - [Pandas functions](#pandas-functions)
    - [Dplyr equivalents](#dplyr-equivalents)

# Python Cheat-sheet

## Pandas functions
 

## Dplyr equivalents

### Add or replace column(s) 
*R*  

    mutate(col_name = transformation_function_call)
    if_else( cond, true_val, false_val )
    case_when( cond1 ~ val1, cond2 ~ val2, TRUE ~ else_val )

Example  

    mutate( ID = str_pad(ID, 7, "left", pad="0"),
            Term_Reporting_Year = as.integer(Term_Reporting_Year),
            Code = coalesce( Code, "UNKNOWN" ),
            Found = 'Y' )

    mutate( Living_Arrangement = if_else( coalesce(HOME,'N') == 'Y', "HOME", "AWAY"),
            Income_Range = case_when(
                Total_Family_Income <=  30000 ~ 1,
                Total_Family_Income <=  48000 ~ 2,
                Total_Family_Income <=  75000 ~ 3,
                Total_Family_Income <= 110000 ~ 4,
                Total_Family_Income >  110000 ~ 5,
                TRUE ~ 99
         ))

*Pandas*  

    .assign

Example  

    .assign(column_name=lambda df: df.Term_ID.str[4:],
           Term_Index=lambda df: df.Term_ID.str[4:].map(term_indexes)
          )

    .assign(ID=lambda df: df.ID.str.zfill(7),
            Code=lambda df: coalesce(df.Code,"UNKNOWN"),
            Found='Y',
        )

    .assign(Living_Arrangement=lambda df: np.where(df.HOME=='Y', "HOME", "AWAY"))

    .assign(Income_Range=lambda df: pd.cut(df['Total_Family_Income'],
                                            [-np.inf, 30000, 48000, 75000, 110000, np.inf],
                                            labels=[1,2,3,4,5],
                                            right=True))

    .assign(ID=lambda df: df.Income_Range.str.zfill(99))
        
### Apply arbitrary functions
*R*  

    Not possible as far as I know

*Pandas*  

    .pipe(func)

Example  

    .pipe(dfcsv, "ipeds_cohorts_1.csv", index=False)

    .pipe((pd.merge, "left"), right=terms, on="Term_ID", how="inner")

    .pipe(dfcsv, "cs_acyr_1.csv", index=False)

    .pipe(dfreplace, ["HOME","AWAY"], rvalue='N')

### Change column types
*R*  

    mutate with 'as' functions
     
Example  

    mutate(Term_Start_Date = as.Date(Term_Start_Date))
 
*Pandas*  

    .astype(col_types_dict) 
    
Example  

    .astype({"Term_Reporting_Year":int})

    .astype({"Term_ID":"str", "Term_Start_Date":"datetime64", "Term_End_Date":"datetime64", "Term_Index":int})
    
### Delete/Drop columns
*R*  
    select(-c(column_list)) 
    
*Pandas*  

    .drop(column_list, axis=1)
   
Example  

    .drop(['EffectiveDatetime'], axis=1)

### Filter columns                  
*R*  

    filter(filter_str) 
    
*Pandas*  

    .loc[lambda df: conditions_using_df] 

Example  

    .loc[lambda df: (df.Cohort_Year == report_year) & (df.Term_ID == report_term)]

### Joining dataframes
*R*  

    left_join(df1, df2) 
    
*Pandas*  

    .pipe((pd.merge, "left"), right=df2, on=keys_list, how="left") 
   
Example  

    .pipe((pd.merge,"left"), right=cs_comp, on=["ID", "Term_Reporting_Year", "EffectiveDatetime"], how="left")

### Pivot data long to wide
*R*  

    pivot_wider

Example  

    pivot_wider( id_cols = c(ID,Term_Reporting_Year,Total_Family_Income), 
                 names_from = Code,
                 values_from = Found
    )

*Pandas*  

    .pivot_table(index=index_list, columns=column_name, values=value_col_name, aggfunc=aggregation_function)

Example  

    .pivot_table(index=["ID","Term_Reporting_Year","Total_Family_Income"],
                 columns="Code",
                 values="Found",
                 aggfunc="first"
                )

### Remove duplicates
*R*  
    distinct()

*Pandas*  

    .drop_duplicates()

### Rename columns
*R*  

    rename(new_col=old_col) 
    
*Pandas*  

    .rename(columns=col_dict) 

Example  

    .rename(columns={"Term_Reporting_Year" : "Cohort_Year",
                     "Term_Start_Date"     : "Cohort_Start_Date" 
                    })

### Select columns
*R*  

    select

Example  

    select( ID, Term_Reporting_Year, Total_Family_Income, Income_Range, Living_Arrangement )

*Pandas*  

    .loc[]

Example  

    .loc[lambda df: df.Term_ID.str[4:].isin(['FA','SP','SU'])]

    .loc[:, ["ID", "Term_Reporting_Year", "Total_Family_Income", "Income_Range", "Living_Arrangement"]]

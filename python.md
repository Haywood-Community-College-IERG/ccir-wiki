- [Python Cheat-sheet](#python-cheat-sheet)
    - [Pandas functions](#pandas-functions)
    - [Dplyr equivalents](#dplyr-equivalents)

# Python Cheat-sheet

## Pandas functions
 

## Dplyr equivalents

Add or replace column(s) 
R: mutate

Pandas:

Can 
assign(column_name=lambda df: df.Term_ID.str[4:],
                Term_Index=lambda df: df.Term_ID.str[4:].map(term_indexes)
            )
            
Select columns
R: select

Pandas
        .loc[lambda df: df.Term_ID.str[4:].isin(['FA','SP','SU'])]
        
 Change column types
 R
     mutate with 'as' functions
     
     Example
         mutate(Term_Start_Date = as.Date(Term_Start_Date))
 
 Pandas
    . astype(col_types_dict) 
    
    Example:
        .astype({"Term_ID":"str", "Term_Start_Date":"datetime64", "Term_End_Date":"datetime64", "Term_Index":int})
     
  

    Apply arbitrary functions
    R
        Not possible
        
    Pandas
       .pipe(func)
       
       Example
           . pipe(dfcsv, "ipeds_cohorts_1.csv", index=False)

           .pipe((pd.merge, "left"), right=terms, on="Term_ID", how="inner")
           
           
    Rename columns
     R
         rename(new_col=old_col) 
         
     Pandas
     
       . rename(columns=col_dict) 
       
       Example
            .rename(columns={"Term_Reporting_Year" : "Cohort_Year",
                             "Term_Start_Date"     : "Cohort_Start_Date" 
                             })
                          
                             
     Filter columns                  
     R
         filter(filter_str) 
         
     Pandas
        .loc[lambda df: conditions_using_df] 
        
        Example   
            .loc[lambda df: (df.Cohort_Year == report_year) & (df.Term_ID == report_term)]
       
        #.pipe(dfcsv, "cs_acyr_1.csv", index=False)

Joining
R
    left_join(df1, df2) 
    
Pandas
   .pipe((pd.merge, "left"), right=df2, on=keys_list, how="left") 
   
   Example
        .pipe((pd.merge,"left"), right=cs_comp, on=["ID", "Term_Reporting_Year", "EffectiveDatetime"], how="left")

Drop columns
R
    select(-c(column_list)) 
    
Pandas
   .drop(column_list, axis=1)
   
   Example
        .drop(['EffectiveDatetime'], axis=1)

        #.pipe(dfcsv, "cs_acyr_2.csv", index=False)

        #R mutate( ID = str_pad(ID, 7, "left", pad="0"),
        #R         Term_Reporting_Year = as.integer(Term_Reporting_Year),
        #R         Code = coalesce( Code, "UNKNOWN" ),
        #R         Found = 'Y' ) %>%
        .astype({"Term_Reporting_Year":int})
        .assign(ID=lambda df: df.ID.str.zfill(7),
                Code=lambda df: coalesce(df.Code,"UNKNOWN"),
                Found='Y',
            )
        #.pipe(dfcsv, "cs_acyr_3.csv", index=False)

        #R pivot_wider( id_cols = c(ID,Term_Reporting_Year,Total_Family_Income), 
        #R              names_from = Code,
        #R              values_from = Found ) %>%
        .pivot_table(index=["ID","Term_Reporting_Year","Total_Family_Income"],
                     columns="Code",
                     values="Found",
                     aggfunc="first")
        .reset_index()
        .pipe(dfreplace, ["HOME","AWAY"], rvalue='N')

        #.pipe(dfcsv, "cs_acyr_4.csv", index=False)

        #R mutate( Living_Arrangement = if_else( coalesce(HOME,'N') == 'Y', "HOME", "AWAY"),
        #R         Income_Range = case_when(
        #R             Total_Family_Income <=  30000 ~ 1,
        #R             Total_Family_Income <=  48000 ~ 2,
        #R             Total_Family_Income <=  75000 ~ 3,
        #R             Total_Family_Income <= 110000 ~ 4,
        #R             Total_Family_Income >  110000 ~ 5,
        #R             TRUE ~ 99
        #R         )) %>%
        .assign(Living_Arrangement=lambda df: np.where(df.HOME=='Y', "HOME", "AWAY"))
        #.pipe(assign_applyfunc, "Income_Range", getIncomeRange, "Total_Family_Income")
        .assign(Income_Range=lambda df: pd.cut(df['Total_Family_Income'],
                                               [-np.inf, 30000, 48000, 75000, 110000, np.inf],
                                               labels=[1,2,3,4,5],
                                               right=True))
        .assign(ID=lambda df: df.Income_Range.str.zfill(99))

        #R select( ID, Term_Reporting_Year, Total_Family_Income, Income_Range, Living_Arrangement ) %>%
        .loc[:, ["ID", "Term_Reporting_Year", "Total_Family_Income", "Income_Range", "Living_Arrangement"]]
        #R distinct()
        .drop_duplicates()

        .pipe(dfcsv, "cs_acyr.csv", index=False)
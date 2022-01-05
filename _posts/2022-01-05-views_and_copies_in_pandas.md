---
layout: post
title: "Views_and_Copies_in_pandas"
subtitle: "explained_with_examples"
date: 2022-01-05 01:56:14
header-style: text
catalog: true
author: "Yuan"
tags: [pandas,python,copy,view]
---
> You are furious because of Ignorant

Copies and views are very common in python, especially when you are working with arrays, lists, pandas. I'm aware of this, still, was trapped in a bug because of this problem when working with pandas.

Basically, there are 2 senarios:
1. Change a value of your df.

   In this case, you need to be careful, making sure your df were changed as wanted, rather than a copy were changed.

2. Subset your current df with slice.
   
   In this case, if a view was assigned to your new df, you will never know what could happen when you made any change to either the original df or the new df.

Example test codes:

```python
import numpy as np
import pandas as pd
np.random.seed(2021)
df = pd.DataFrame(np.random.choice(10, (4, 3)), columns=list('ABC'))
print("Orignal df")
print(df)
#    A  B  C
# 0  4  5  9
# 1  0  6  5
# 2  8  6  6
# 3  6  6  1
print(df.dtypes)

my_slice = df.iloc[1:3, :]
my_slice2 = df.iloc[1:3, ].copy()

print(my_slice)
df.iloc[1, 1] = 3
df
print(my_slice)
print(my_slice2)
print("\nSenario1\n-------------------")
# Senario1
# Never do this!!
# This is a two-step operation.
# First, make a copy = df[df.A > 5]
# Then, get ['B'] from that copy
df[df.A > 5]['B']
# instead, do this:
df.loc[df.A > 5, 'B']
# Example
df[df.A > 5]['B'] = 4  # Not change df
print("after df[][]=xxx, df does not change")
print(df)
df.loc[df.A > 5, 'B'] = 4  # Change df
print("after df.loc[,]=xxx, df changes!!!")
print(df)

# Senario2
print("\nSenario2\n-------------------")
np.random.seed(2021)
df = pd.DataFrame(np.random.choice(10, (4, 3)), columns=list('ABC'))
print("Orignal df:")
print(df)
# Never do this:
# The index is a slice, thus this will return a view
df2no = df.loc[1:3, :]  # Returns a view
df3no = df['B']  # Returns a view
# Instead do this:
df2 = df.loc[1:3, :].copy()  # Returns a copy
df3 = df['B'].copy()
# example
# Changes the values of df2no, but not df2
df.iloc[3, 1] = 2
print("after df changes, df2no:")
print(df2no)
print("after df changes, df2:")
print(df2)
print("after df changes, df3no:")
print(df3no)
print("after df changes, df3:")
print(df3)
# Alternatively, change df2no will change df if the dtype does not change
# Will change the value of df,get warning:Try using .loc[row_indexer,col_indexer] = value instead
print("Before df2no,df3no changes,df:")
print(df)
df2no.iloc[2, 0] = 0
df3no[1] = 0
print("after df2no, df3no changes to another value of the same dtype, df:")
print(df)
df2no.iloc[2, 0] = 'a'  # Will NOT change the value of df
print("after df2no changes to another value of the other dtype, df:")
print(df)
print("after df2no changes to another value of the other dtype, df2no:")
print(df2no)
df3no[1] = 'b'  # Will NOT change the value of df
print("after df3no changes to another value of the other dtype, df:")
print(df)
print("after df3no changes to another value of the other dtype, df3no:")
print(df3no)

```


---

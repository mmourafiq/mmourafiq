---
layout: blog
title: "Generating an excel report with python"
subtitle: "Pandas pivot tables to an excel sheet."
cover_image: images/blog/excel-python.png
cover_image_caption: ""
tags: [data, python, pandas, excel]
---
For many data analysts and business people excel is a powerful tool for reporting. But very often excel reports become cumbersome and difficult to extend, especially when it comes to gathering data from several sources. In this post we will generate an excel report using python (pandas and openpyxl). We will also look briefly at a way to create an excel template and then generate an excel report with several sheets based on this template.

We will be looking at some startups investments data.

```python
import pandas as pd
import numpy as np

df = pd.read_csv('investments.csv', sep=';')
```

To make the report simple, we will be using only a subset of the columns in the csv file, and we will look at the data from 2007 for funding rounds of type: angel, seed, venture, private equity, and undisclosed.

```python
df1 = df[['funded_year', 'funded_quarter', 'funding_round_type', 'raised_amount_usd']]
df1 = df1[df.funded_year >= 2007]
df1 = df1[df1.funding_round_type.isin(['angel', 'private_equity', 'seed', 'venture', 'undisclosed'])]
```

The data look like this:

|funded_year 	  |funded_quarter 	|funding_round_type 	|raised_amount_usd 	  |
|---------------|-----------------|---------------------|---------------------|
|2014           |2014-Q3          |venture              |5 956 174            |
|2014           |2014-Q3          |private_equity       |81 216 295           |
|2014           |2014-Q3          |seed                 |1 300 000            |
|2014           |2014-Q3          |seed                 |NaN                  |
|2014           |2014-Q2          |seed                 |500 000              |
|2014           |2014-Q3          |venture              |2 500 000            |
|2014           |2014-Q3          |venture              |7 000 000            |
|...            |...              |...                  |...                  |

We notice that the data contains some undefined values, so we have to do some cleaning

```python
df1 = df1.fillna(0)
def to_int(x):
    try:
        x = int(x.replace(' ', ''))
    except (AttributeError, ValueError):
        return 0

df1.raised_amount_usd = df1.raised_amount_usd.map(to_int)
df1.raised_amount_usd = df1.raised_amount_usd.astype(int)
```

Now let's create our pivot tables

```python
table1 = pd.pivot_table(df1, index=['funding_round_type'],
               columns=['funded_year', 'funded_quarter'],
               values=['raised_amount_usd'],
               aggfunc=[lambda x: len(x)],fill_value=0)

table2 = pd.pivot_table(df1, index=['funding_round_type'],
               columns=['funded_year', 'funded_quarter'],
               values=['raised_amount_usd'],
               aggfunc=[np.sum],fill_value=0)
```

This will generate a 2 pivot tables similar to this:


<div>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="15" halign="left">&lt;count&gt;</th>
    </tr>
    <tr>
      <th></th>
      <th colspan="15" halign="left">raised_amount_usd</th>
    </tr>
    <tr>
      <th>funded_year</th>
      <th colspan="4" halign="left">2007</th>
      <th colspan="3" halign="left">2008</th>
      <th>...</th>
      <th colspan="3" halign="left">2013</th>
      <th colspan="4" halign="left">2014</th>
    </tr>
    <tr>
      <th>funded_quarter</th>
      <th>2007-Q1</th>
      <th>2007-Q2</th>
      <th>2007-Q3</th>
      <th>2007-Q4</th>
      <th>2008-Q1</th>
      <th>2008-Q2</th>
      <th>...</th>
      <th>2013-Q2</th>
      <th>2013-Q3</th>
      <th>2013-Q4</th>
      <th>2014-Q1</th>
      <th>2014-Q2</th>
      <th>2014-Q3</th>
    </tr>
    <tr>
      <th>funding_round_type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>angel</th>
      <td>97</td>
      <td>67</td>
      <td>50</td>
      <td>71</td>
      <td>122</td>
      <td>69</td>
      <td>...</td>
      <td>161</td>
      <td>226</td>
      <td>141</td>
      <td>211</td>
      <td>123</td>
      <td>77</td>
    </tr>
    <tr>
      <th>private_equity</th>
      <td>16</td>
      <td>14</td>
      <td>22</td>
      <td>10</td>
      <td>13</td>
      <td>13</td>
      <td>...</td>
      <td>244</td>
      <td>187</td>
      <td>132</td>
      <td>101</td>
      <td>131</td>
      <td>68</td>
    </tr>
    <tr>
      <th>seed</th>
      <td>110</td>
      <td>88</td>
      <td>92</td>
      <td>111</td>
      <td>219</td>
      <td>143</td>
      <td>...</td>
      <td>1870</td>
      <td>2368</td>
      <td>2097</td>
      <td>1922</td>
      <td>1686</td>
      <td>1763</td>
    </tr>
    <tr>
      <th>undisclosed</th>
      <td>395</td>
      <td>268</td>
      <td>255</td>
      <td>243</td>
      <td>362</td>
      <td>354</td>
      <td>...</td>
      <td>710</td>
      <td>454</td>
      <td>647</td>
      <td>830</td>
      <td>304</td>
      <td>98</td>
    </tr>
    <tr>
      <th>venture</th>
      <td>1294</td>
      <td>1207</td>
      <td>1297</td>
      <td>1144</td>
      <td>1440</td>
      <td>1434</td>
      <td>...</td>
      <td>1856</td>
      <td>2065</td>
      <td>2147</td>
      <td>2263</td>
      <td>3019</td>
      <td>2905</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 32 columns</p>
</div>

Now let's generate an excel report with 2 sheets containing the results of the pivot tables

```python
writer = pd.ExcelWriter('report.xlsx')
table1.to_excel(writer, 'Sheet1')
table2.to_excel(writer, 'Sheet2')
writer.save()
```

The result output

![excel_report](/images/blog/report.png)

To go a bit further, let's imagine that we want to generate the report from a template. For this we will need the python module openpyxl, because it's the only python module that allows to modify an excel file.

The first step is to create a template for our report.

![excel_report](/images/blog/report-template.png)

You can notice that we named an excel range `values` to be able to update the values in that range easily from the python side. Here's an utility function to update an excel sheet from python given a named range or cell range.

```python
import openpyxl
from openpyxl import load_workbook

def update_range(worksheet, data, cell_range=None, named_range=None):
    """
    Updates an excel worksheet with the given data.
    :param worksheet: an excel worksheet
    :param data: data used to update the worksheet cell range (list, tuple, np.ndarray, pd.Dataframe)
    :param cell_range: a string representing the cell range, e.g. 'AB12:XX23'
    :param named_range: a string representing an excel named range
    """

    def clean_data(data):
        if not isinstance(data, (list, tuple, np.ndarray, pd.DataFrame)):
            raise TypeError('Invalid data, data should be an array type iterable.')

        if not len(data):
            raise ValueError('You need to provide data to update the cells')

        if isinstance(data, pd.DataFrame):
            data = data.values

        elif isinstance(data, (list, tuple)):
            data = np.array(data)

        return np.hstack(data)

    def clean_cells(worksheet, cell_range, named_range):
        # check that we can access a cell range
        if not any((cell_range, named_range) or all((cell_range, named_range))):
            raise ValueError('`cell_range` or `named_range` should be provided.')

        # get the cell range
        if cell_range:
            try:
                cells = np.hstack(worksheet[cell_range])
            except (CellCoordinatesException, AttributeError):
                raise ValueError('The cell range provided is invalid, cell range must be in the form XX--[:YY--]')

        else:
            try:
                cells = worksheet.get_named_range(named_range)
            except (NamedRangeException, TypeError):
                raise ValueError('The current worksheet {} does not contain any named range {}.'.format(
                    worksheet.title,
                    named_range))

        # checking that we have cells to update, and data
        if not len(cells):
            raise ValueError('You need to provide cells to update.')

        return cells

    cells = clean_cells(worksheet, cell_range, named_range)
    data = clean_data(data)

    # check that the data has the same dimension as cells
    if len(cells) != data.size:
        raise ValueError('Cells({}) should have the same dimension as the data({}).'.format(len(cells), data.size))

    for i, cell in enumerate(cells):
        cell.value = data[i]
```

Now we are ready to generate our first report from the template

```python
import openpyxl
from openpyxl import load_workbook

# load the excel template
workbook = load_workbook('template.xlsx')

# number of funding rounds by year
table1 = pd.pivot_table(df1, index=['funding_round_type'],
               columns=['funded_year'],
               values=['raised_amount_usd'],
               aggfunc=[lambda x: len(x)], fill_value=0)

worksheet = workbook.active
worksheet.title = 'count_by_year'
update_range(workbook.active, table1.values, named_range='values')
# create a new excel report
workbook.save('report1.xlsx')
```

This will keep our template unmodified and generate a report with one excel sheet `count_by_year` containing the values from the pivot table.

![excel_report](/images/blog/excel-report-count.png)

The last part of this part is to show a work around when we need to generate a report with multiple sheets based on the same template. Things get a bit complicated, because no python module allow to copy/clone a sheet and update it, I managed to find a way to do this by reloading the workbook each time we copy the template sheet, but I hope that this option will be supported by the openpyxl module in the future, for further reference there's a [ticket](https://bitbucket.org/openpyxl/openpyxl/issues/171/copy-worksheet-function) for this.

```python
import openpyxl
from openpyxl import load_workbook
from copy import copy

# load the excel template
workbook = load_workbook('template.xlsx')
# create a new excel report
workbook.save('report2.xlsx')


# number of funding rounds by year
table1 = pd.pivot_table(df1, index=['funding_round_type'],
               columns=['funded_year'],
               values=['raised_amount_usd'],
               aggfunc=[lambda x: len(x)], fill_value=0)

# sum of funding rounds by year
table2 = pd.pivot_table(df1, index=['funding_round_type'],
              columns=['funded_year'],
              values=['raised_amount_usd'],
              aggfunc=[np.sum], fill_value=0)

def create_sheet(wb_name, sheet_name, values):
    workbook = load_workbook(wb_name)
    worksheet = workbook.active
    # clone the template worksheet
    update_range(workbook.active, values, named_range='values')
    # add worksheet to the workbook
    worksheet.title = sheet_name
    workbook._add_sheet(copy.copy(worksheet))
    # since we can't use deepcopy, we need to save the workbook and reload it
    # otherwise we will be updating the same references of all copied sheets
    workbook.save(wb_name)

create_sheet('report2.xlsx', 'count_by_year', table1.values)
create_sheet('report2.xlsx', 'sum_by_year', table2.values)
# we need to remove the current template sheet
workbook = load_workbook('report2.xlsx')
workbook.remove_sheet(workbook.active)
workbook.save('report2.xlsx')
```

The output result

![excel_report](/images/blog/excel-report-count-sum.png)


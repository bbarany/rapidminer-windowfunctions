# Window functions in RapidMiner

An implementation of some window functions as a [RapidMiner](https://rapidminer.com/) process.

_Balázs Bárány <balazs@tud.at>_

## About window functions

Window functions in SQL databases are aggregation functions applied to defined
"partitions" or groups of data, without actually grouping records together. The
PostgreSQL project has a good 
[introduction](https://www.postgresql.org/docs/current/static/tutorial-window.html)
to window functions. 

This very useful functionality of modern databases has been missing in
RapidMiner until now. Advanced users were of course able to implement similar
processes on their own, but it's not trivial. So I created this project to
provide a central implementation of window functions as an easy-to-use
RapidMiner process.

The [Series extension](https://marketplace.rapidminer.com/UpdateServer/faces/product_details.xhtml?productId=rmx_series)
contains some operators called Window. They are useful for time series data but
not generally applicable to arbitrary datasets. 

## About this project

The goal is to create a process that can be used in RapidMiner processes as a
"blackbox", calculating new attributes for attributes given in the process
parameters. The process is available here, it can be saved in the RapidMiner
repository and used without changing it.

The scope is to include useful functionality as far as possible in native
RapidMiner, without any extensions or programming (aside from the internal
Groovy scripting). 

The license is Apache 2.0, so you can use it without any restrictions in your
processes and also change it. If you implement a cool new function, please send
me you version or put it on Github and send me a pull request.

### Available Functionality

All functions available in the RapidMiner 
[Aggregate](http://docs.rapidminer.com/studio/operators/blending/table/grouping/aggregate.html) 
operator are also available in the Window Functions process. (This operator is
used in the background.)

In SQL databases, partial/cumulative aggregation is possible by specifying both
PARTITION BY and ORDER BY in the window function. This gives you running sums,
for example. This is not yet implemented in the Window Functions project, but it
should be possible.

Some well-known SQL window functions are not available in RapidMiner's Aggregate
operator. The following functions have been implemented here:

- **row_number**: Unique sequential numbering of rows inside a group, ordered by
  an arbitrary attribute. Ties (equal values in the ordering attribute) still
  produce new row numbers, so this is quite useful for genereting an ID-like
  attribute for each group.

- **rank**: Numerical rank of rows inside a group, ordered by an attribute. Ties
  result in the same rank (e. g. 1, 1, 3, 4, 5). The numbering can contain gaps
  behind ties, e. g. there is no second rank if two examples have rank = 1.

- **dense_rank**: Numerical rank of rows inside a group, ordered by an
  attribute. Ties result in the same rank but the numbering doesn't contain any
  gaps (e. g. 1, 1, 2, 3, 3, 4, 5). 

More functions could be implemented in the process.

### Examples

The example processes are available from the [**examples** directory](examples/). 

- Calculating the average temperature for each different value of "outlook" and
adding a global rank by humidity on the Golf dataset:

![Golf with average temperature and humidity rank](images/golf-temperature-humidity.png)

- Calculating the average value of "a3" in the Iris dataset and filtering examples
that fall into the range of 90 to 110 % of their group:

![Iris with average temperature by group](images/iris-average-filter.png)


## Usage

Import the Window Functions process into your repository by downloading and
saving it in your Local Repository folder. You can also save it anywhere and use
_File/Import Process..._ to open it in Studio and save it in the repository of
your choice, for example on a Server.

Drag the process from the repository into your process or use Execute Process
and select its location. In any case, you need to specify a few macros in the
Execute Process operator to configure it.

#### Configuration macros

**groupFields**: (required) Comma-separated list of attributes to group on. You
can put spaces after the commas. At least one grouping attribute is required. If
only one attribute is used, is has to be nominal (this is a RapidMiner
limitation). 

Sometimes it's useful to work without grouping, e. g. to calculate an overall
average or a rank by some attribute. In this case, just generate a dummy
grouping attribute with a value like "x", use it for grouping, and remove it
later. (An obvious enhancement idea for a future version: the process could do
this automatically.)

**function**: (required) Name of the function to apply. Either one of the function names
from the Aggregate operator or one of the implemented functions:

- row_number
- rank 
- dense_rank

**valueField**: (required) The name of the value attribute which the aggregation function
acts upon. 

**orderField**: For the ranking and numbering functions, the ordering attribute
inside the subgroup. If you're using the standard aggregation functions of
RapidMiner's Aggregate operator, this doesn't need to be specified.

## Limitations

The process can be slow if the number of groups is high. This is especially the
case when using the custom functions (row_number, rank etc.). The reason is that
these are implemented with Groovy scripts that will be executed many times in a loop. 

Some useful functions defined in the SQL standard are not yet implemented:
percent_rank, cume_dist, ntile, lag, lead, first_value, last_value, nth_value.

(lag and lead are available in the Series extension.)

Windows can only be specified with attribute values but not in the other ways
specified in SQL. (ROWS BETWEEN ... PRECEDING ... FOLLOWING)

The ordering attribute is only used for the ranking and row_number functions.
The built-in aggregation functions are always calculated for the entire group,
regardless of the **orderField** setting. 

The **groupFields** attribute has to be nominal if only one is used. Numerical
values are converted to nominal, which should be OK but could cause some
surprises in special cases.

There is no support (yet) for working without a grouping field (which is
possible in SQL with OVER ()). The workaround is to add a constant grouping
field before executing the Window Functions process and remove it after.

### Possible enhancements

* Checking functionality in the Series extension that could be useful here.
(However, the process is supposed to be able to run without any installed
extensions.)

* Implementing more functions:
  * Standard SQL functions mentioned in the Limitations
  * cumsum for cumulated sum (ordered by an attribute)
  * min/max until current record (SQL: MIN(value) OVER (ORDER BY whatever))

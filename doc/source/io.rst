.. _io:

.. currentmodule:: pandas

.. ipython:: python
   :suppress:

   import numpy as np
   np.random.seed(123456)
   from pandas import *
   from StringIO import StringIO
   import pandas.util.testing as tm
   randn = np.random.randn
   np.set_printoptions(precision=4, suppress=True)
   import matplotlib.pyplot as plt
   plt.close('all')

*******************************
IO Tools (Text, CSV, HDF5, ...)
*******************************

CSV & Text files
----------------

The two workhorse functions for reading text files (a.k.a. flat files) are
:func:`~pandas.io.parsers.read_csv` and :func:`~pandas.io.parsers.read_table`.
They both use the same parsing code to intelligently convert tabular
data into a DataFrame object. They can take a number of arguments:

  - ``path_or_buffer``: Either a string path to a file, or any object with a
    ``read`` method (such as an open file or ``StringIO``).
  - ``delimiter``: For ``read_table`` only, a regular expression to split
    fields on. ``read_csv`` uses the ``csv`` module to do this and hence only
    supports comma-separated values.
  - ``header``: row number to use as the column names, and the start of the data.
    Defaults to 0 (first row); specify None if there is no header row.
  - ``names``: List of column names to use if header is None.
  - ``skiprows``: A collection of numbers for rows in the file to skip.
  - ``index_col``: column number, or list of column numbers, to use as the
    ``index`` (row labels) of the resulting DataFrame. By default, it will number
    the rows without using any column, unless there is one more data column than
    there are headers, in which case the first column is taken as the index.
  - ``parse_dates``: If True, attempt to parse the index column as dates. False
    by default.
  - ``date_parser``: function to use to parse strings into datetime
    objects. If ``parse_dates`` is True, it defaults to the very robust
    ``dateutil.parser``. Specifying this implicitly sets ``parse_dates`` as True.
  - ``na_values``: optional list of strings to recognize as NaN (missing values),
    in addition to a default set.
  

.. code-block:: ipython

    In [1]: print open('foo.csv').read()
    date,A,B,C
    20090101,a,1,2
    20090102,b,3,4
    20090103,c,4,5
    
    # A basic index is created by default:
    In [3]: read_csv('foo.csv')
    Out[3]:
       date      A  B  C
    0  20090101  a  1  2
    1  20090102  b  3  4
    2  20090103  c  4  5

    # Use a column as an index, and parse it as dates.
    In [3]: df = read_csv('foo.csv', index_col=0, parse_dates=True)
    
    In [4]: df
    Out[4]:
                A  B  C
    2009-01-01  a  1  2
    2009-01-02  b  3  4
    2009-01-03  c  4  5

    # These are python datetime objects
    In [16]: df.index
    Out[16]: Index([2009-01-01 00:00:00, 2009-01-02 00:00:00,
                    2009-01-03 00:00:00], dtype=object)


The parsers make every attempt to "do the right thing" and not be very
fragile. Type inference is a pretty big deal. So if a column can be coerced to
integer dtype without altering the contents, it will do so. Any non-numeric
columns will come through as object dtype as with the rest of pandas objects.

Reading DataFrame objects with ``MultiIndex``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Suppose you have data indexed by two columns:

.. ipython:: python

   print open('data/mindex_ex.csv').read()

The ``index_col`` argument to ``read_csv`` and ``read_table`` can take a list of
column numbers to turn multiple columns into a ``MultiIndex``:

.. ipython:: python

   df = read_csv("data/mindex_ex.csv", index_col=[0,1])
   df
   df.ix[1978]

Excel 2003 files
----------------

The ``ExcelFile`` class can read an Excel 2003 file using the ``xlrd`` Python
module and use the same parsing code as the above to convert tabular data into
a DataFrame. To use it, create the ``ExcelFile`` object:

.. code-block:: python

   xls = ExcelFile('path_to_file.xls')

Then use the ``parse`` instance method with a sheetname, then use the same
additional arguments as the parsers above:

.. code-block:: python

   xls.parse('Sheet1', index_col=None, na_values=['NA'])

HDF5 (PyTables)
---------------

``HDFStore`` is a dict-like object which reads and writes pandas to the high
performance HDF5 format using the excellent `PyTables
<http://www.pytables.org/>`__ library.

.. ipython:: python
   :suppress:

   import os
   os.remove('store.h5')

.. ipython:: python

   store = HDFStore('store.h5')
   print store

Objects can be written to the file just like adding key-value pairs to a dict:

.. ipython:: python

   index = DateRange('1/1/2000', periods=8)
   s = Series(randn(5), index=['a', 'b', 'c', 'd', 'e'])
   df = DataFrame(randn(8, 3), index=index,
                  columns=['A', 'B', 'C'])
   wp = Panel(randn(2, 5, 4), items=['Item1', 'Item2'],
              major_axis=DateRange('1/1/2000', periods=5),
              minor_axis=['A', 'B', 'C', 'D'])

   store['s'] = s
   store['df'] = df
   store['wp'] = wp
   store

In a current or later Python session, you can retrieve stored objects:

.. ipython:: python

   store['df']

Storing in Table format
~~~~~~~~~~~~~~~~~~~~~~~

Querying objects stored in Table format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. ipython:: python
   :suppress:

   store.close()
   import os
   os.remove('store.h5')

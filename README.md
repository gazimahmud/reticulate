
R Interface to Python
=====================

[![Travis-CI Build Status](https://travis-ci.org/rstudio/reticulate.svg?branch=master)](https://travis-ci.org/rstudio/reticulate) [![Appveyor Build Status](https://ci.appveyor.com/api/projects/status/github/rstudio/reticulate?svg=true)](https://ci.appveyor.com/project/rstudio/reticulate) [![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/reticulate)](https://cran.r-project.org/package=reticulate)

The **reticulate** package provides a comprehensive set of Python interoperability tools for R.

If you are an R developer that uses Python for some of your work or a member of data science team that uses both languages, reticulate can dramatically streamline your workflow. Reticulte embeds a Python session within your R session, enabling the following types of interoperability:

1.  [Python in R Markdown](#python-in-r-markdown) --- A new Python language engine for R Markdown that supports bi-directional communication between R and Python (R chunks can access Python objects and vice-versa).

2.  [Sourcing Python scripts](#sourcing-python-scripts) --- The `source_python()` function enables you to source a Python script the same way you would `source()` an R script (Python functions and objects defined within the script become directly available to the R session).

3.  [Importing Python modules](#importing-python-modules) -- The `import()` function enables you to import any Python module and call it's functions directly from R.

4.  [Python REPL](#python-repl) --- The `py_repl()` function creates a Python REPL (Read-Eval-Print-Loop") within R. Objects you create within the Python REPL are available to your R session (and vice-versa).

Each of these techniques will be explained in more detail below.

### Getting Started

First, install **reticulate** from GitHub as follows:

``` r
devtools::install_github("rstudio/reticulate")
```

Then, review the examples below and check out the additional articles enumerated in the [Learning More](#learning-more) section.

Python in R Markdown
--------------------

The **reticulate** package includes a Python engine for [R Markdown](http://rmarkdown.rstudio.com) with the following features:

<img src="images/rmarkdown_engine.png" class="screenshot" align="right" style="margin-left: 40px;" width="400" />

1.  Run Python chunks in a single Python session embedded within your R session (shared variables/state between Python chunks)

2.  Printing of Python output, including graphical output from [matplotlib](https://matplotlib.org/).

3.  Access to objects created within Python chunks from R using the `py` object (e.g. `py$x` would access an `x` variable created within Python from R).

4.  Access to objects created within R chunks from Python using the `r` object (e.g. `r.x` would access to `x` variable created within R from Python)

Built in conversion for many Python object types is provided, including [NumPy](http://www.numpy.org/) arrays and [Pandas](https://pandas.pydata.org/) data frames. From example, you can use Pandas to read and manipulate data then easily plot the Pandas data frame using [ggplot2](http://ggplot2.org/) (as illustrated above).

The reticulate Python engine is enabled by default within R Markdown whenever reticulate is installed. See the `eng_python()` documentation for additional details on the R Markdown Python engine.

Sourcing Python scripts
-----------------------

You can source any Python script just as you would source an R script using the `source_python()` function. For example, if you had the following Python script *flights.py*:

``` python
import pandas
def read_flights(file):
  flights = pandas.read_csv("flights.csv")
  flights = flights[flights['dest'] == "ORD"]
  flights = flights[['carrier', 'dep_delay', 'arr_delay']]
  flights = flights.dropna()
  return flights
```

Then you can source the script and call the `read_flights()` function as follows:

``` r
source_python("flights.py")
flights <- read_flights("flights.csv")

library(ggplot2)
ggplot(flights, aes(carrier, arr_delay)) + geom_point() + geom_jitter()
```

See the `source_python()` documentation for additional details on sourcing Python code.

Importing Python modules
------------------------

You can use the `import()` function to import any Python module and call it from R. For example, this code imports the Python `os` module and calls some functions within it:

``` r
library(reticulate)
os <- import("os")
os$listdir(".")
```

     [1] ".git"             ".gitignore"       ".Rbuildignore"    ".RData"          
     [5] ".Rhistory"        ".Rproj.user"      ".travis.yml"      "appveyor.yml"    
     [9] "DESCRIPTION"      "docs"             "external"         "index.html"      
    [13] "index.Rmd"        "inst"             "issues"           "LICENSE"         
    [17] "man"              "NAMESPACE"        "NEWS.md"          "pkgdown"         
    [21] "R"                "README.md"        "reticulate.Rproj" "src"             
    [25] "tests"            "vignettes"      

Functions and other data within Python modules and classes can be accessed via the `$` operator (analogous to the way you would interact with an R list, environment, or reference class).

Python REPL
-----------

If you want to work with Python interactively you can call the `py_repl()` function, which provides a Python REPL embedded within your R session. Objects created within the Python REPL can be accessed from R using the `py` object exported from reticulate. For example:

<img src="images/python_repl.png" class="screenshot" />

Enter `exit` within the Python REPL to return to the R prompt.

Note that Python code can also access objects from within the R session using the `r` object:

<img src="images/python_r_object.png" class="screenshot" />

See the `py_repl()` documentation for additional details on using the embedded Python REPL.

Type conversions
----------------

When calling into Python, R data types are automatically converted to their equivalent Python types. When values are returned from Python to R they are converted back to R types. Types are converted as follows:

| R                      | Python            | Examples                                         |
|------------------------|-------------------|--------------------------------------------------|
| Single-element vector  | Scalar            | `1`, `1L`, `TRUE`, `"foo"`                       |
| Multi-element vector   | List              | `c(1.0, 2.0, 3.0)`, `c(1L, 2L, 3L)`              |
| List of multiple types | Tuple             | `list(1L, TRUE, "foo")`                          |
| Named list             | Dict              | `list(a = 1L, b = 2.0)`, `dict(x = x_data)`      |
| Matrix/Array           | NumPy ndarray     | `matrix(c(1,2,3,4), nrow = 2, ncol = 2)`         |
| Data Frame             | Pandas DataFrame  | `data.frame(x = c(1,2,3), y = c("a", "b", "c"))` |
| Function               | Python function   | `function(x) x + 1`                              |
| NULL, TRUE, FALSE      | None, True, False | `NULL`, `TRUE`, `FALSE`                          |

If a Python object of a custom class is returned then an R reference to that object is returned. You can call methods and access properties of the object just as if it was an instance of an R reference class.

Learning more
-------------

The following articles cover the various aspects of using **reticulate**:

-   [Calling Python from R](https://rstudio.github.io/reticulate/articles/calling_python.html) --- Describes the various ways to access Python objects from R as well as functions available for more advanced interactions and conversion behavior.

-   [Python Version Configuration](https://rstudio.github.io/reticulate/articles/versions.html) --- Describes facilities for determining which version of Python is used by reticulate within an R session.

-   [Using reticulate in an R Package](https://rstudio.github.io/reticulate/articles/package.html) --- Guidelines and best practices for using reticulate in an R package.

-   [Arrays in R and Python](https://rstudio.github.io/reticulate/articles/arrays.html) --- Advanced discussion of the differences between arrays in R and Python and the implications for conversion and interoperability.

Why reticulate?
---------------

From [Wikipedia](https://en.wikipedia.org/wiki/Reticulated_python):

> The reticulated python is a speicies of python found in Southeast Asia. They are the world's longest snakes and longest reptiles...The specific name, reticulatus, is Latin meaning "net-like", or reticulated, and is a reference to the complex colour pattern.

The reticulate package enables a new flavor of "reticulated" Python code that is weaved closely together with R.




# The CRAN database

The CRAN database provides an API for programatically accessing all
meta-data of CRAN R packages. This API can be used for various purposes,
here are three examples I am woking on right now:
* Writing a package manager for R. The package manager can use the
  CRAN DB API to query dependencies, or other meta data.
* Building a search engine for CRAN packages. The DB itself does not
  provide a search API, but it can be (easily) mirrored in a search
  engine.
* Creating an RSS feed for the new, updated or archived packages on CRAN.

## The `crandb` API

This is the API provided in the `crandb` R package.
This is still under planning and development.

## The raw API

This is the raw JSON API.

We will use the
[`httr`](https://github.com/hadley/httr) package to query it, and the
[`jsonlite`](https://github.com/jeroenooms/jsonlite) package to nicely
format it. [`magrittr`](https://github.com/smbache/magrittr) is loaded
for the pipes. We use `DB` to query the API and format the result.


```r
library(magrittr)
skip_lines <- function(text, head = 1e6, tail = 1e6) {
	text <- strsplit(text, "\n")[[1]]
	tail <- min(tail, max(0, length(text) - head))
	skip_text <- if (length(text) > head + tail) {
		paste("\n... not showing", length(text) - head - tail, "lines ...\n")
	} else {
		character()
	}
    c(head(text, head), skip_text, tail(text, tail)) %>%
		paste(collapse = "\n")
}
DB <- function(api, head = 1e6, tail = head) {
  paste0("http://db.r-pkg.org", "/", api) %>%
    httr::GET() %>%
	httr::content(as = "text", encoding = "UTF-8") %>%
	jsonlite::prettify() %>%
	skip_lines(head = head, tail = tail) %>%
	cat()
}
```

### `/:pkg` Latest version of a package

The result includes all fields verbatim from the DESCRIPTION file,
plus some extra:
* `date`: The date and time when the package was published on CRAN.
  This is needed, because especially old packages might not have some
  (or any) of the `Date`, `Date/Publication` or `Packaged` fields.
* `releases`: The R version(s) that were released when this version
  of the package was the latest on CRAN.
* The `Suggests`, `Depends`, etc. fields are formatted as named lists.


```r
DB("/magrittr")
```

```
## {
## 	"Package" : "magrittr",
## 	"Type" : "Package",
## 	"Title" : "magrittr - a forward-pipe operator for R",
## 	"Version" : "1.0.1",
## 	"Date" : "2014-05-14",
## 	"Author" : "Stefan Milton Bache <stefan@stefanbache.dk> and\u000aHadley Wickham <h.wickham@gmail.com>",
## 	"Maintainer" : "Stefan Milton Bache <stefan@stefanbache.dk>",
## 	"Description" : "Provides a mechanism for chaining commands with a\u000anew forward-pipe operator. Ceci n'est pas un pipe.",
## 	"Suggests" : {
## 		"testthat" : "*",
## 		"knitr" : "*"
## 	},
## 	"VignetteBuilder" : "knitr",
## 	"License" : "MIT + file LICENSE",
## 	"Packaged" : "2014-05-15 18:49:49 UTC; shb",
## 	"NeedsCompilation" : "no",
## 	"Repository" : "CRAN",
## 	"Date/Publication" : "2014-05-15 21:12:27",
## 	"crandb_file_date" : "2014-05-15 15:12:29",
## 	"date" : "2014-05-15T21:12:27+00:00",
## 	"releases" : [
## 		"3.1.1"
## 	]
## }
```

### `/:pkg/:version` A specific version of a package

The format is the same as for the latest version.


```r
DB("/magrittr/1.0.0")
```

```
## {
## 	"Package" : "magrittr",
## 	"Type" : "Package",
## 	"Title" : "magrittr - a forward-pipe operator for R",
## 	"Version" : "1.0.0",
## 	"Date" : "2014-01-19",
## 	"Author" : "Stefan Milton Bache <stefan@stefanbache.dk> and\u000aHadley Wickham <h.wickham@gmail.com>",
## 	"Maintainer" : "Stefan <stefan@stefanbache.dk>",
## 	"Description" : "Provides a mechanism for chaining commands with a\u000anew forward-pipe operator. Ceci n'est pas un pipe.",
## 	"Suggests" : {
## 		"testthat" : "*",
## 		"knitr" : "*"
## 	},
## 	"VignetteBuilder" : "knitr",
## 	"License" : "MIT + file LICENSE",
## 	"Packaged" : "2014-02-25 16:05:59 UTC; shb",
## 	"NeedsCompilation" : "no",
## 	"Repository" : "CRAN",
## 	"Date/Publication" : "2014-02-25 18:01:09",
## 	"crandb_file_date" : "2014-02-25 12:01:11",
## 	"date" : "2014-02-25T18:01:09+00:00",
## 	"releases" : [
## 		"3.0.3",
## 		"3.1.0"
## 	]
## }
```

### `/:pkg/all` All versions of a package

The result is a list of package versions in the `versions` field, in the
format above, plus some extra:
* `name`: The name of the package.
* `title`: The title field of the package.
* `latest`: The latest version of the package.
* `archived`: Whether the package was archived.
* `timeline`: All versions and their release dates of the package.


```r
DB("/magrittr/all")
```

```
## {
## 	"_id" : "magrittr",
## 	"_rev" : "1-a8d52d0e409af456cc4c5eac957b5609",
## 	"name" : "magrittr",
## 	"versions" : {
## 		"1.0.0" : {
## 			"Package" : "magrittr",
## 			"Type" : "Package",
## 			"Title" : "magrittr - a forward-pipe operator for R",
## 			"Version" : "1.0.0",
## 			"Date" : "2014-01-19",
## 			"Author" : "Stefan Milton Bache <stefan@stefanbache.dk> and\u000aHadley Wickham <h.wickham@gmail.com>",
## 			"Maintainer" : "Stefan <stefan@stefanbache.dk>",
## 			"Description" : "Provides a mechanism for chaining commands with a\u000anew forward-pipe operator. Ceci n'est pas un pipe.",
## 			"Suggests" : {
## 				"testthat" : "*",
## 				"knitr" : "*"
## 			},
## 			"VignetteBuilder" : "knitr",
## 			"License" : "MIT + file LICENSE",
## 			"Packaged" : "2014-02-25 16:05:59 UTC; shb",
## 			"NeedsCompilation" : "no",
## 			"Repository" : "CRAN",
## 			"Date/Publication" : "2014-02-25 18:01:09",
## 			"crandb_file_date" : "2014-02-25 12:01:11",
## 			"date" : "2014-02-25T18:01:09+00:00",
## 			"releases" : [
## 				"3.0.3",
## 				"3.1.0"
## 			]
## 		},
## 		"1.0.1" : {
## 			"Package" : "magrittr",
## 			"Type" : "Package",
## 			"Title" : "magrittr - a forward-pipe operator for R",
## 			"Version" : "1.0.1",
## 			"Date" : "2014-05-14",
## 			"Author" : "Stefan Milton Bache <stefan@stefanbache.dk> and\u000aHadley Wickham <h.wickham@gmail.com>",
## 			"Maintainer" : "Stefan Milton Bache <stefan@stefanbache.dk>",
## 			"Description" : "Provides a mechanism for chaining commands with a\u000anew forward-pipe operator. Ceci n'est pas un pipe.",
## 			"Suggests" : {
## 				"testthat" : "*",
## 				"knitr" : "*"
## 			},
## 			"VignetteBuilder" : "knitr",
## 			"License" : "MIT + file LICENSE",
## 			"Packaged" : "2014-05-15 18:49:49 UTC; shb",
## 			"NeedsCompilation" : "no",
## 			"Repository" : "CRAN",
## 			"Date/Publication" : "2014-05-15 21:12:27",
## 			"crandb_file_date" : "2014-05-15 15:12:29",
## 			"date" : "2014-05-15T21:12:27+00:00",
## 			"releases" : [
## 				"3.1.1"
## 			]
## 		}
## 	},
## 	"timeline" : {
## 		"1.0.0" : "2014-02-25T18:01:09+00:00",
## 		"1.0.1" : "2014-05-15T21:12:27+00:00"
## 	},
## 	"latest" : "1.0.1",
## 	"title" : "magrittr - a forward-pipe operator for R",
## 	"archived" : false
## }
```

### `/-/all` All packages, in alphabetical order

Note that this API point _really_ returns a list of all active CRAN packages
(currently about 6,000), so it is a good idea to query the date in
chunks. `limit` specifies the number of records to return, and `startkey`
can be used to specify the first key to list. Note that the result will
include the full records of the package, all package versions.


```r
DB("/-/all?limit=3", head = 20)
```

```
## {
## 	"A3" : {
## 		"_id" : "A3",
## 		"_rev" : "1-bb4704a78f23aaa57cd8f4bc524f12e6",
## 		"name" : "A3",
## 		"versions" : {
## 			"0.9.1" : {
## 				"Package" : "A3",
## 				"Type" : "Package",
## 				"Title" : "A3: Accurate, Adaptable, and Accessible Error Metrics for\u000aPredictive Models",
## 				"Version" : "0.9.1",
## 				"Date" : "2013-02-06",
## 				"Author" : "Scott Fortmann-Roe",
## 				"Maintainer" : "Scott Fortmann-Roe <scottfr@berkeley.edu>",
## 				"Description" : "This package supplies tools for tabulating and analyzing\u000athe results of predictive models. The methods employed are\u000aapplicable to virtually any predictive model and make\u000acomparisons between different methodologies straightforward.",
## 				"License" : "GPL (>= 2)",
## 				"Depends" : {
## 					"xtable" : "*",
## 					"pbapply" : "*"
## 				},
## 
## ... not showing 459 lines ...
## 
## 					"3.0.0",
## 					"3.0.1",
## 					"3.0.2",
## 					"3.0.3",
## 					"3.1.0",
## 					"3.1.1"
## 				]
## 			}
## 		},
## 		"timeline" : {
## 			"0.1" : "2011-11-05T10:48:08+00:00",
## 			"0.2" : "2012-01-22T20:21:13+00:00",
## 			"0.3" : "2012-06-28T17:02:35+00:00",
## 			"0.4" : "2012-09-15T13:13:26+00:00"
## 		},
## 		"latest" : "0.4",
## 		"title" : "ABCDE_FBA: A-Biologist-Can-Do-Everything of Flux Balance\u000aAnalysis with this package.",
## 		"archived" : false
## 	}
## }
```

### `/-/latest` Latest versions of all packages

This is similar to `/-/all`, but only the latest versions of
the packages are returned, instead of the complete records with
all versions.


```r
DB("/-/latest?limit=3", head = 20)
```

```
## {
## 	"A3" : {
## 		"Package" : "A3",
## 		"Type" : "Package",
## 		"Title" : "A3: Accurate, Adaptable, and Accessible Error Metrics for\u000aPredictive Models",
## 		"Version" : "0.9.2",
## 		"Date" : "2013-03-24",
## 		"Author" : "Scott Fortmann-Roe",
## 		"Maintainer" : "Scott Fortmann-Roe <scottfr@berkeley.edu>",
## 		"Description" : "This package supplies tools for tabulating and analyzing\u000athe results of predictive models. The methods employed are\u000aapplicable to virtually any predictive model and make\u000acomparisons between different methodologies straightforward.",
## 		"License" : "GPL (>= 2)",
## 		"Depends" : {
## 			"R" : ">= 2.15.0",
## 			"xtable" : "*",
## 			"pbapply" : "*"
## 		},
## 		"Suggests" : {
## 			"randomForest" : "*",
## 			"e1071" : "*"
## 		},
## 
## ... not showing 58 lines ...
## 
## 		"Description" : "Functions for Constraint Based Simulation using Flux\u000aBalance Analysis and informative analysis of the data generated\u000aduring simulation.",
## 		"License" : "GPL-2",
## 		"Lazyload" : "yes",
## 		"Packaged" : "2012-09-15 11:23:36 UTC; Abhilash",
## 		"Repository" : "CRAN",
## 		"Date/Publication" : "2012-09-15 13:13:26",
## 		"crandb_file_date" : "2012-09-15 09:13:27",
## 		"date" : "2012-09-15T13:13:26+00:00",
## 		"releases" : [
## 			"2.15.2",
## 			"2.15.3",
## 			"3.0.0",
## 			"3.0.1",
## 			"3.0.2",
## 			"3.0.3",
## 			"3.1.0",
## 			"3.1.1"
## 		]
## 	}
## }
```

### `/-/desc` Short description of latest package versions

Latest versions of all packages, in alphabetical order. It also
contains the `title` fields of the packages. Only active (not archived)
packages are included.


```r
DB("/-/desc?limit=5")
```

```
## {
## 	"A3" : {
## 		"version" : "0.9.2",
## 		"title" : "A3: Accurate, Adaptable, and Accessible Error Metrics for\u000aPredictive Models"
## 	},
## 	"abc" : {
## 		"version" : "2.0",
## 		"title" : "Tools for Approximate Bayesian Computation (ABC)"
## 	},
## 	"abcdeFBA" : {
## 		"version" : "0.4",
## 		"title" : "ABCDE_FBA: A-Biologist-Can-Do-Everything of Flux Balance\u000aAnalysis with this package."
## 	},
## 	"ABCExtremes" : {
## 		"version" : "1.0",
## 		"title" : "ABC Extremes"
## 	},
## 	"ABCoptim" : {
## 		"version" : "0.13.11",
## 		"title" : "Implementation of Artificial Bee Colony (ABC) Optimization"
## 	}
## }
```

### `/-/allall` Complete records for all packages, including archived ones

This is similar to `/-/all`, but lists the archived packages as well.


```r
DB("/-/desc?limit=2", head = 20)
```

```
## {
## 	"A3" : {
## 		"version" : "0.9.2",
## 		"title" : "A3: Accurate, Adaptable, and Accessible Error Metrics for\u000aPredictive Models"
## 	},
## 	"abc" : {
## 		"version" : "2.0",
## 		"title" : "Tools for Approximate Bayesian Computation (ABC)"
## 	}
## }
```

### `/-/pkgreleases` Package releases

All versions of all packages, in the order of their release. Note that
this includes each version of each package separately, so it is a very
long list, and it is a good idea to use the `limit` parameter. `descending`
can be used to reverse the ordering, and start with the most recent
releases.


```r
DB("/-/pkgreleases?limit=3&descending=true", head = 20)
```

```
## [
## 	{
## 		"date" : "2014-09-02T19:44:02+00:00",
## 		"name" : "rappdirs",
## 		"event" : "released",
## 		"package" : {
## 			"Package" : "rappdirs",
## 			"Type" : "Package",
## 			"Title" : "Application directories: determine where to save data, caches\u000aand logs.",
## 			"Version" : "0.3",
## 			"Authors@R" : "c(\u000aperson(\"Hadley\", \"Wickham\", email = \"h.wickham@gmail.com\",\u000arole = c(\"trl\", \"cre\", \"cph\")),\u000aperson(\"RStudio\", role = \"cph\"),\u000aperson(\"Sridhar\", \"Ratnakumar\", role = \"aut\"),\u000aperson(\"Trent\", \"Mick\", role = \"aut\"),\u000aperson(\"ActiveState\", role = \"cph\", comment =\u000a\"R/appdir.r, R/cache.r, R/data.r, R/log.r translated from appdirs\"),\u000aperson(\"Eddy\", \"Petrisor\", role = \"ctb\"),\u000aperson(\"Trevor\", \"Davis\", role = c(\"trl\", \"aut\")),\u000aperson(\"Gabor\", \"Csardi\", role = \"ctb\"),\u000aperson(\"Gregory\", \"Jefferis\", role = \"ctb\")\u000a)",
## 			"Depends" : {
## 				"R" : ">= 2.14",
## 				"methods" : "*"
## 			},
## 			"Suggests" : {
## 				"testthat" : "*",
## 				"roxygen2" : "*"
## 			},
## 			"Description" : "An easy way to determine which directories on the users computer\u000ayou should use to save data, caches and logs. A port of Python's Appdirs\u000a(\\url{https://github.com/ActiveState/appdirs}) to R.",
## 
## ... not showing 57 lines ...
## 
## 		"package" : {
## 			"Package" : "ica",
## 			"Type" : "Package",
## 			"Title" : "Independent Component Analysis",
## 			"Version" : "1.0-0",
## 			"Date" : "2014-09-02",
## 			"Author" : "Nathaniel E. Helwig <helwig@umn.edu>",
## 			"Maintainer" : "Nathaniel E. Helwig <helwig@umn.edu>",
## 			"Description" : "Independent Component Analysis (ICA) using various algorithms: FastICA, Information-Maximization (Infomax), and Joint Approximate Diagonalization of Eigenmatrices (JADE).",
## 			"License" : "GPL (>= 2)",
## 			"Packaged" : "2014-09-02 16:26:17 UTC; Nate",
## 			"NeedsCompilation" : "no",
## 			"Repository" : "CRAN",
## 			"Date/Publication" : "2014-09-02 19:41:15",
## 			"crandb_file_date" : "2014-09-02 14:36:34",
## 			"date" : "2014-09-02T19:41:15+00:00",
## 			"releases" : []
## 		}
## 	}
## ]
```

### `/-/archivals` Package archivals

Package archival events, sorted by date times. The latest version
of the package record is also included. Again, use the `limit` parameter
to query in chunks, and the `descending` parameter to reverse the order,
and see most recent archivals last.


```r
DB("/-/archivals?limit=3&descending=true", head = 20)
```

```
## [
## 	{
## 		"date" : "2014-09-02T13:31:18+00:00",
## 		"name" : "MIfuns",
## 		"event" : "archived",
## 		"package" : {
## 			"Package" : "MIfuns",
## 			"Type" : "Package",
## 			"Title" : "Pharmacometric tools for data preparation, modeling, simulation,\u000aand reporting",
## 			"Version" : "5.1",
## 			"Date" : "2011-09-07",
## 			"Author" : "Metrum Institute (http://metruminstitute.org): Bill Knebel,\u000aLeonid Gibianski, Tim Bergsma",
## 			"Maintainer" : "Tim Bergsma <timb@metrumrg.com>",
## 			"Depends" : {
## 				"reshape" : "*",
## 				"methods" : "*",
## 				"lattice" : "*",
## 				"grid" : "*",
## 				"XML" : "*",
## 				"MASS" : "*"
## 
## ... not showing 74 lines ...
## 
## 			},
## 			"Description" : "A stub package to ease transition to 'parallel'.\u000aIt imports from 'parallel' or 'tools' and re-exports most of the\u000afunctionality formerly in package 'multicore'.\u000aThis will be removed from CRAN during 2014.",
## 			"License" : "GPL-2",
## 			"crandb_file_date" : "2014-05-17 05:43:01",
## 			"OS_type" : "unix",
## 			"Repository" : "CRAN",
## 			"Date/Publication" : "2014-05-17 11:42:59",
## 			"Imports" : {
## 				"parallel" : "*",
## 				"tools" : "*"
## 			},
## 			"Packaged" : "2014-05-17 09:39:13 UTC; ripley",
## 			"NeedsCompilation" : "no",
## 			"date" : "2014-05-17T11:42:59+00:00",
## 			"releases" : [
## 				"3.1.1"
## 			]
## 		}
## 	}
## ]
```

### `/-/events` Release and archival events

The union of `/-/pkgreleases` and `/-/archivals`, so it includes
all package releases and archivals.


```r
DB("/-/events?limit=3&descending=true", head = 20)
```

```
## [
## 	{
## 		"date" : "2014-09-02T19:44:02+00:00",
## 		"name" : "rappdirs",
## 		"event" : "released",
## 		"package" : {
## 			"Package" : "rappdirs",
## 			"Type" : "Package",
## 			"Title" : "Application directories: determine where to save data, caches\u000aand logs.",
## 			"Version" : "0.3",
## 			"Authors@R" : "c(\u000aperson(\"Hadley\", \"Wickham\", email = \"h.wickham@gmail.com\",\u000arole = c(\"trl\", \"cre\", \"cph\")),\u000aperson(\"RStudio\", role = \"cph\"),\u000aperson(\"Sridhar\", \"Ratnakumar\", role = \"aut\"),\u000aperson(\"Trent\", \"Mick\", role = \"aut\"),\u000aperson(\"ActiveState\", role = \"cph\", comment =\u000a\"R/appdir.r, R/cache.r, R/data.r, R/log.r translated from appdirs\"),\u000aperson(\"Eddy\", \"Petrisor\", role = \"ctb\"),\u000aperson(\"Trevor\", \"Davis\", role = c(\"trl\", \"aut\")),\u000aperson(\"Gabor\", \"Csardi\", role = \"ctb\"),\u000aperson(\"Gregory\", \"Jefferis\", role = \"ctb\")\u000a)",
## 			"Depends" : {
## 				"R" : ">= 2.14",
## 				"methods" : "*"
## 			},
## 			"Suggests" : {
## 				"testthat" : "*",
## 				"roxygen2" : "*"
## 			},
## 			"Description" : "An easy way to determine which directories on the users computer\u000ayou should use to save data, caches and logs. A port of Python's Appdirs\u000a(\\url{https://github.com/ActiveState/appdirs}) to R.",
## 
## ... not showing 57 lines ...
## 
## 		"package" : {
## 			"Package" : "ica",
## 			"Type" : "Package",
## 			"Title" : "Independent Component Analysis",
## 			"Version" : "1.0-0",
## 			"Date" : "2014-09-02",
## 			"Author" : "Nathaniel E. Helwig <helwig@umn.edu>",
## 			"Maintainer" : "Nathaniel E. Helwig <helwig@umn.edu>",
## 			"Description" : "Independent Component Analysis (ICA) using various algorithms: FastICA, Information-Maximization (Infomax), and Joint Approximate Diagonalization of Eigenmatrices (JADE).",
## 			"License" : "GPL (>= 2)",
## 			"Packaged" : "2014-09-02 16:26:17 UTC; Nate",
## 			"NeedsCompilation" : "no",
## 			"Repository" : "CRAN",
## 			"Date/Publication" : "2014-09-02 19:41:15",
## 			"crandb_file_date" : "2014-09-02 14:36:34",
## 			"date" : "2014-09-02T19:41:15+00:00",
## 			"releases" : []
## 		}
## 	}
## ]
```

### `/-/releases` List of R versions

List of all R versions supported by the database. Currently this goes
back to version 2.0.0, older versions will be potentially added later.


```r
DB("/-/releases", head = 20)
```

```
## [
## 	{
## 		"version" : "2.0.0",
## 		"date" : "2004-10-04T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "2.0.1",
## 		"date" : "2004-11-15T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "2.1.0",
## 		"date" : "2005-04-18T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "2.1.1",
## 		"date" : "2005-06-20T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "2.2.0",
## 		"date" : "2005-10-06T00:00:00+00:00"
## 
## ... not showing 146 lines ...
## 
## 		"version" : "3.0.1",
## 		"date" : "2013-05-16T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "3.0.2",
## 		"date" : "2013-09-25T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "3.0.3",
## 		"date" : "2014-03-06T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "3.1.0",
## 		"date" : "2014-04-10T00:00:00+00:00"
## 	},
## 	{
## 		"version" : "3.1.1",
## 		"date" : "2014-07-10T00:00:00+00:00"
## 	}
## ]
```

### `/-/releasepkgs/:version` Packages that were current at an R release

All package records that were current when a given version of R
was released. This is essentially a snapshot of CRAN, for a given R
release.


```r
DB("/-/releasepkgs/2.15.3", head = 20)
```

```
## {
## 	"A3" : {
## 		"Package" : "A3",
## 		"Type" : "Package",
## 		"Title" : "A3: Accurate, Adaptable, and Accessible Error Metrics for\u000aPredictive Models",
## 		"Version" : "0.9.1",
## 		"Date" : "2013-02-06",
## 		"Author" : "Scott Fortmann-Roe",
## 		"Maintainer" : "Scott Fortmann-Roe <scottfr@berkeley.edu>",
## 		"Description" : "This package supplies tools for tabulating and analyzing\u000athe results of predictive models. The methods employed are\u000aapplicable to virtually any predictive model and make\u000acomparisons between different methodologies straightforward.",
## 		"License" : "GPL (>= 2)",
## 		"Depends" : {
## 			"xtable" : "*",
## 			"pbapply" : "*"
## 		},
## 		"Suggests" : {
## 			"randomForest" : "*",
## 			"e1071" : "*"
## 		},
## 		"Packaged" : "2013-02-06 16:46:12 UTC; scott",
## 
## ... not showing 167076 lines ...
## 
## 		"Depends" : {
## 			"R" : ">= 2.4.0",
## 			"Kendall" : "*"
## 		},
## 		"Suggests" : {},
## 		"Description" : "The zyp package contains an efficient implementation of\u000aSen's slope method plus implementation of Xuebin Zhang's\u000a(Zhang, 1999) and Yue-Pilon's (Yue, 2002) prewhitening\u000aapproaches to determining trends in climate data.",
## 		"License" : "LGPL-2.1",
## 		"URL" : "http://www.r-project.org",
## 		"Packaged" : "2012-10-29 09:00:03 UTC; ripley",
## 		"Repository" : "CRAN",
## 		"Date/Publication" : "2012-10-29 09:00:03",
## 		"crandb_file_date" : "2012-10-29 04:00:03",
## 		"date" : "2012-10-29T09:00:03+00:00",
## 		"releases" : [
## 			"2.15.3",
## 			"3.0.0",
## 			"3.0.1"
## 		]
## 	}
## }
```

### `/-/release/:version` Package versions that were current at an R release

Similar to the previous list, but it only includes the version numbers,
and not the records of the packages.


```r
DB("/-/release/2.15.3", head = 20)
```

```
## {
## 	"A3" : "0.9.1",
## 	"aaMI" : "1.0-1",
## 	"abc" : "1.6",
## 	"abcdeFBA" : "0.4",
## 	"abctools" : "0.1-2",
## 	"abd" : "0.2-4",
## 	"abind" : "1.4-0",
## 	"aBioMarVsuit" : "1.0",
## 	"abn" : "0.82",
## 	"AcceptanceSampling" : "1.0-2",
## 	"ACCLMA" : "1.0",
## 	"accuracy" : "1.35",
## 	"ACD" : "1.0-0",
## 	"Ace" : "0.0.8",
## 	"acepack" : "1.3-3.2",
## 	"acer" : "0.1.2",
## 	"aCGH.Spline" : "2.2",
## 	"ACNE" : "0.5.0",
## 	"acs" : "0.8",
## 
## ... not showing 4918 lines ...
## 
## 	"YjdnJlp" : "0.9.8",
## 	"YourCast" : "1.5-1",
## 	"YuGene" : "1.0",
## 	"ZeBook" : "0.3",
## 	"Zelig" : "4.1-3",
## 	"ZeligChoice" : "0.7-0",
## 	"ZeligGAM" : "0.7-0",
## 	"ZeligMultilevel" : "0.7-0",
## 	"zendeskR" : "0.3",
## 	"zic" : "0.7.5",
## 	"zicounts" : "1.1.5",
## 	"ZIGP" : "3.8",
## 	"zipcode" : "1.0",
## 	"zipfR" : "0.6-6",
## 	"zmatrix" : "1.1",
## 	"zoeppritz" : "1.0-3",
## 	"zoo" : "1.7-9",
## 	"zooimage" : "3.0-3",
## 	"zyp" : "0.9-1"
## }
```

### `/-/releasedesc/:version` Short description of CRAN snapshots

Similar to `/-/release`, but it also include the the `title` fields of
the packages.


```r
DB("/-/releasedesc/2.15.3", head = 20)
```

```
## {
## 	"A3" : {
## 		"version" : "0.9.1",
## 		"title" : "A3: Accurate, Adaptable, and Accessible Error Metrics for\u000aPredictive Models"
## 	},
## 	"aaMI" : {
## 		"version" : "1.0-1",
## 		"title" : "Mutual information for protein sequence alignments"
## 	},
## 	"abc" : {
## 		"version" : "1.6",
## 		"title" : "Tools for Approximate Bayesian Computation (ABC)"
## 	},
## 	"abcdeFBA" : {
## 		"version" : "0.4",
## 		"title" : "ABCDE_FBA: A-Biologist-Can-Do-Everything of Flux Balance\u000aAnalysis with this package."
## 	},
## 	"abctools" : {
## 		"version" : "0.1-2",
## 		"title" : "Algorithms for ABC summary statistics selection"
## 
## ... not showing 19766 lines ...
## 
## 		"version" : "1.1",
## 		"title" : "Zero-offset matrices"
## 	},
## 	"zoeppritz" : {
## 		"version" : "1.0-3",
## 		"title" : "Zoeppritz Equations"
## 	},
## 	"zoo" : {
## 		"version" : "1.7-9",
## 		"title" : "S3 Infrastructure for Regular and Irregular Time Series (Z's\u000aordered observations)"
## 	},
## 	"zooimage" : {
## 		"version" : "3.0-3",
## 		"title" : "Analysis of numerical zooplankton images"
## 	},
## 	"zyp" : {
## 		"version" : "0.9-1",
## 		"title" : "Zhang + Yue-Pilon trends package"
## 	}
## }
```

### `/-/topdeps/:version` Packages most depended upon

Top twenty packages. It includes all forms of dependencies, and it can be
restricted to dependencies that were in place at a given R release. 


```r
DB("/-/topdeps/3.1.1")
```

```
## [
## 	{
## 		"MASS" : 805
## 	},
## 	{
## 		"lattice" : 414
## 	},
## 	{
## 		"ggplot2" : 310
## 	},
## 	{
## 		"Matrix" : 289
## 	},
## 	{
## 		"mvtnorm" : 280
## 	},
## 	{
## 		"testthat" : 272
## 	},
## 	{
## 		"survival" : 261
## 	},
## 	{
## 		"Rcpp" : 238
## 	},
## 	{
## 		"plyr" : 235
## 	},
## 	{
## 		"knitr" : 166
## 	},
## 	{
## 		"XML" : 162
## 	},
## 	{
## 		"rgl" : 155
## 	},
## 	{
## 		"nlme" : 152
## 	},
## 	{
## 		"sp" : 150
## 	},
## 	{
## 		"igraph" : 143
## 	},
## 	{
## 		"coda" : 135
## 	},
## 	{
## 		"boot" : 133
## 	},
## 	{
## 		"RCurl" : 128
## 	},
## 	{
## 		"RUnit" : 119
## 	},
## 	{
## 		"reshape2" : 117
## 	}
## ]
```

For the latest versions of the packages and their dependencies, you can set
`:version` to `dev`:


```r
DB("/-/topdeps/devel")
```

```
## [
## 	{
## 		"MASS" : 835
## 	},
## 	{
## 		"lattice" : 421
## 	},
## 	{
## 		"ggplot2" : 328
## 	},
## 	{
## 		"Matrix" : 310
## 	},
## 	{
## 		"testthat" : 308
## 	},
## 	{
## 		"mvtnorm" : 290
## 	},
## 	{
## 		"survival" : 271
## 	},
## 	{
## 		"Rcpp" : 267
## 	},
## 	{
## 		"plyr" : 253
## 	},
## 	{
## 		"knitr" : 207
## 	},
## 	{
## 		"XML" : 168
## 	},
## 	{
## 		"sp" : 168
## 	},
## 	{
## 		"rgl" : 163
## 	},
## 	{
## 		"nlme" : 156
## 	},
## 	{
## 		"igraph" : 148
## 	},
## 	{
## 		"RCurl" : 142
## 	},
## 	{
## 		"coda" : 142
## 	},
## 	{
## 		"boot" : 138
## 	},
## 	{
## 		"reshape2" : 129
## 	},
## 	{
## 		"stringr" : 123
## 	}
## ]
```

### `/-/deps/:version` Number of reverse dependencies

For all packages, not just the top twenty.


```r
DB("/-/deps/2.15.1", head = 20)
```

```
## {
## 	"abind" : 38,
## 	"accuracy" : 4,
## 	"acepack" : 2,
## 	"aCGH" : 2,
## 	"actuar" : 9,
## 	"ada" : 3,
## 	"adabag" : 1,
## 	"adapt" : 2,
## 	"ade4" : 25,
## 	"ade4TkGUI" : 1,
## 	"adegenet" : 4,
## 	"adehabitat" : 4,
## 	"adehabitatHR" : 1,
## 	"adehabitatLT" : 2,
## 	"adehabitatMA" : 3,
## 	"adephylo" : 1,
## 	"ADGofTest" : 4,
## 	"adimpro" : 3,
## 	"adlift" : 1,
## 
## ... not showing 1462 lines ...
## 
## 	"xlsxjars" : 1,
## 	"XML" : 98,
## 	"XMLRPC" : 1,
## 	"XMLSchema" : 3,
## 	"xpose4" : 1,
## 	"xpose4classic" : 1,
## 	"xpose4data" : 4,
## 	"xpose4generic" : 3,
## 	"xpose4specific" : 2,
## 	"xtable" : 72,
## 	"xtermStyle" : 1,
## 	"xts" : 25,
## 	"yacca" : 1,
## 	"yaImpute" : 1,
## 	"YaleToolkit" : 2,
## 	"yaml" : 1,
## 	"Zelig" : 5,
## 	"zipfR" : 2,
## 	"zoo" : 76
## }
```

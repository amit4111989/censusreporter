About Census Reporter
=====================

The United States Census Bureau provides a massive amount of data about the American people, covering topics from demographics to poverty rates to educational attainment, and at geographical levels ranging from the entire country down to city blocks. As a product of the federal government, the <a href="http://factfinder2.census.gov/faces/nav/jsf/pages/index.xhtml">data is free to use</a>. But for working journalists&mdash;especially those who don't have experience with the particulars of Census data&mdash;navigating these datasets on deadline can be a challenge.

Census Reporter's goal is to make it easier for journalists to write stories using Census data. Our focus is on the American Community Survey; we want to help people understand what the survey covers and help them quickly find data from places they care about. Census Reporter received <a href="http://www.niemanlab.org/2012/10/knight-funding-expands-ires-journalist-friendly-census-site/">funding from the Knight News Challenge</a>, and primary development took place from March 2013 through June 2014.

The <a href="http://censusreporter.org/">Census Reporter website</a> includes three primary types of pages: <a href="http://censusreporter.org/profiles/16000US5367000-spokane-wa/">geographical profiles</a>, which provide an overview of data indicators from a particular place; <a href="http://censusreporter.org/data/table/?table=B15002&geo_ids=050|04000US17&primary_geo_id=04000US17">data comparisons</a>, which use tabular, map and distribution formats to show information from a table across a group of geographies; and <a href="http://censusreporter.org/topics/income/">topical overviews</a>, which document the concepts and tables the ACS uses to cover specific subject areas.

In This Guide
=============

* [Setting up for local development](#setting-up-for-local-development)
* [Getting data from our API (the basics)](#getting-data-from-our-api-the-basics)
    * [Show data](#show-data)

Setting up for local development
================================

Census Reporter is an open-source project, so not only is the data free to use, so is the code. Developers in South Africa forked this repository to build <a href="http://wazimap.co.za/">Wazi</a>, for example, a site exploring South African data. We'd love it if you'd fork this repository, too, and maybe you even have features you'd like to contribute back!

Here's what you need to know to get a local version of Census Reporter up and running. These instructions assume you're using <a href="http://virtualenv.readthedocs.org/en/latest/">virtualenv</a> and <a href="http://virtualenvwrapper.readthedocs.org/en/latest/">virtualenvwrapper</a> to manage your development environments.

First, clone this repository to your machine and move into your new project directory:

    >> git clone git@github.com:censusreporter/censusreporter.git
    >> cd <your cloned repo dir>

Create the virtual environment for your local project, activate it and install the required libraries:

    >> mkvirtualenv census --no-site-packages
    >> workon census
    >> pip install -r requirements.txt

If you've upgraded XCode on OS X Mavericks, you may well see some compilation errors here. If so, try this:

    >> ARCHFLAGS=-Wno-error=unused-command-line-argument-hard-error-in-future pip install -r requirements.txt

With your development environment still active, make sure it has the path settings it will need:

    >> add2virtualenv ./censusreporter
    >> add2virtualenv ./censusreporter/apps

And make sure your development environment knows the proper DJANGO_SETTINGS_MODULE by creating a `postactivate` script ...

    >> cdvirtualenv bin
    >> touch postactivate

... and then using your favorite editor to add these two lines to your new `postactivate` script:

    export DJANGO_SETTINGS_MODULE='config.dev.settings'
    echo "DJANGO_SETTINGS_MODULE set to $DJANGO_SETTINGS_MODULE"

Save and close the file. Reactivate your development environment so `postactivate` gets triggered, then fire up your local Census Reporter:

    >> deactivate
    >> workon census
    >> cd <your cloned repo dir>
    >> python manage.py runserver

Hurray! Your local copy of Census Reporter will be hitting our production data sources, so you should be able to search and browse just like you would on the live website. If you'd like to go a step further, you can <a href="https://github.com/censusreporter/census-api">run the API locally</a>, and even <a href="http://censusreporter.tumblr.com/post/73727555158/easier-access-to-acs-data">load the entire Postgres database locally</a>, as well. But if you're primarily interested in adding features to the Census Reporter website, the instructions above are likely all you'll need to get going.

Getting data from our API (the basics)
======================================

As part of the Census Reporter project, we've loaded ACS data into a Postgres database to make queries significantly easier. The Census Reporter website gets information from this database using the API at api.censusreporter.org. We'll provide extensive API documentation separately, but here are the basic endpoints you're likely to use:

###Show data
This endpoint does the heavy lifting for Census Reporter's profile and comparison pages. Given a release code, a table code, and a geography, it will return American Community Survey data. A common call to this endpoint might look like:

    http://api.censusreporter.org/1.0/data/show/latest?table_ids=B01001&geo_ids=16000US5367000

This will return data for Spokane, WA, from the "Sex By Age" table, using the "latest" ACS release available. In this case, "latest" determines not only the year of release, but also the estimate used. The ACS provides <a href="http://www.census.gov/acs/www/guidance_for_data_users/estimates/">three datasets per year</a>: the 1-year, which uses 12 months of data to arrive at estimates for areas with at least 65,000 residents; the 3-year, which uses 36 months of data and covers areas with at least 20,000 people; and the 5-year, which uses 60 months of data and covers areas of all sizes.

For this API endpoint, "latest" is a shortcut that asks for the most current estimate from the most recent release year, favoring 1-year over 3-year over 5-year data. Because Spokane, WA, has more than 65,000 residents, the API call above would return data from the ACS 2012 1-year release.

You can ask for a specific release and estimate by exchanging "latest" for a release code that looks like `acs{year}_{estimate}yr`. So a call to:

    http://api.censusreporter.org/1.0/data/show/acs2012_5yr?table_ids=B01001&geo_ids=16000US5367000

... would return data for Spokane, WA, from the "Sex By Age" table, using the ACS 2012 5-year release.

You can ask for multiple tables at a time by passing a comma-separated `table_ids` list:

    http://api.censusreporter.org/1.0/data/show/latest?table_ids=B01001,B01002&geo_ids=16000US5367000

And you can ask for multiple geographies by passing a comma-separated `geo_ids` list:

    http://api.censusreporter.org/1.0/data/show/latest?table_ids=B01001,B01002&geo_ids=16000US5367000,16000US1714000

One particularly common use case is to request data for all geographies of a particular class within a particular parent geography, e.g. "compare all counties in Washington state." Identifying each county's geoID individually would be unwieldy, so we added a shortcut for this type of request. Your comma-separated list of `geo_ids` can contain one or more items that use the pipe character to describe a comparison set like this: `{child_summary_level}|{parent_geoid}`.

The Census uses summary levels to identify classes of geographies (like counties or school districts or census tracts), and each one <a href="http://censusreporter.org/topics/geography/">is represented by a three-digit code</a>. For this API endpoint, "all counties in Washington state" can be represented as `050|04000US53`, so a request to:

    http://api.censusreporter.org/1.0/data/show/latest?table_ids=B01001&geo_ids=050|04000US53

... would return "Sex By Age" data for all 39 counties in Washington. Great? Great! But hold on, let's talk about this for a second.

Specifying "latest" as the release parameter means we're asking for the most current estimate from the most recent release. Although some of the counties in Washington state are large enough to be in the 1-year release, many are not. Several counties fall into the 3-year release, and several more are small enough to only exist in the 5-year release. Data from different estimates should never be compared, so the API will return data from the lowest common denominator: in this case, the 2012 5-year release.

You won't have to guess which release you're getting data from, though; the API response from this endpoint will include four objects:

* `release`: metadata about the ACS release that provided the data in this response
* `tables`: metadata, including title and column information, about the tables requested
* `data`: the actual data for the geographies requested, including estimates and margins of error, nested according to geoID > table code > estimate|error > column code
* `geography`: metadata, including geoID and name, about the geographies requested

The entire response for the "all counties in Washington" example above <a href="https://gist.github.com/ryanpitts/71517ac65333c72ccb8e">looks like this</a>; below is an excerpt so you can see what to expect.

    {
        "release": {
            "id": "acs2012_5yr",
            "name": "ACS 2012 5-year",
            "years": "2008-2012"
        },
        "tables": {
            "B01001": {
                "title": "Sex by Age",
                "universe": "Total Population",
                "denominator_column_id": "B01001001",
                "columns": {
                    "B01001001": {
                        "name": "Total:",
                        "indent": 0
                    },
                    "B01001002": {
                        "name": "Male:",
                        "indent": 1
                    },
                    ...
                    "B01001049": {
                        "name": "85 years and over",
                        "indent": 2
                    }
                }
            }
        },
        "data": {
            "05000US53001": {
                "B01001": {
                    "estimate": {
                        "B01001001": 18575,
                        "B01001002": 9453,
                        ...
                        "B01001049": 214
                    },
                    "error": {
                        "B01001001": 0,
                        "B01001002": 24,
                        ...
                        "B01001049": 53
                    }
                }
            }
        },
        "geography": {
            "04000US53": {
                "name": "Washington"
            },
            "05000US53047": {
                "name": "Okanogan County, WA"
            },
            ...
            "05000US53035": {
                "name": "Kitsap County, WA"
            }
        }
    }

Note that Washington state's data is also included in this API response. When you ask for a set of geographies&mdash;in this case, 050|04000US53&mdash;the API will automatically include the parent geography's data as well, so you can perform comparisons.

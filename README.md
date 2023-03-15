# Community Reports
Building reports from the Augur database schema is an important way for catalogouing questions that our users ask, as well as how we can make the most of the extensive, validated data Augur gathers. Everything is not "dashboard material". Over time, reports people find useful will become APIS in Augur that can be used to construct automated community reports. This is a repository for engaged requirements gathering in the agile spirit. 

The template directory is where we begin to understand what the reports "are", as we expect individual users to create their own directories where they make new reports, work with us to create new reports, and then share them with the CHAOSS community. The goal is to then build generalized augur queries/APIs, etc, and make them available as tools. 

## Identifying Repository ID's

After finishing setup, [use the START_HERE.ipynb file to identify the Repository ID's available in your database.](./START_HERE.ipynb). You will, in most cases, be interested in a subset of all available repositories at any given moment, and this notebook lets you grab the ID info you will need to run those reports.

# Setup
## Prerequisites
1. Python 3.x
2. pip
3. virtualenv package `pip3 install virtualenv`
4. Install `geckodriver` for your platform if you want to write annotated PNG files out. This is a great way to automate report generation!
    - osx: `brew install geckodriver`
    - Linux, Windows: Download the latest geckodriver release for your platform from `https://github.com/mozilla/geckodriver/releases` and follow installation instructions. You can also get source code from that link. 

## Setup augur-community-reports
1. Fork the augur-community-reports repository
2. Clone your fork of the repository locally
```
git clone https://github.com/<your-fork>/augur-community-reports
````
Typically, your fork will take this form: ` git clone https://github.com/<your-github-username>/augur-community-reports`, the only exception being if you create your fork under a GitHub organization. In that case, you would replace `your-github-username` with the name of the organization where you created the fork. 

3. Create your python virtual environment wherever you routinely store them. We use a `virtualenvs` directory. 
```
virtualenv --python=python3 virtualenvs/augur-community-reports
```
4. Activate your virtual environment
```
source  ./virtualenvs/augur-community-reports/bin/activate
```
6. Install the necessary Python libraries for Python 3.9 and later
```
pip install -r requirements.txt 
```
7. Change into the directory of your clone
```
cd augur-community-reports
```
8. Run Jupyter Lab
```
jupyter lab
```

# The information below is for advanced users. 

## Create a read only user on your augur database, like this: 
```
CREATE USER chaoss WITH PASSWORD 'port88';
GRANT CONNECT ON DATABASE augur TO chaoss;
GRANT USAGE ON SCHEMA augur_data TO chaoss;
GRANT USAGE ON SCHEMA spdx TO chaoss;
GRANT USAGE ON SCHEMA augur_operations TO chaoss;
GRANT SELECT ON ALL TABLES IN SCHEMA augur_data TO chaoss;
GRANT SELECT ON ALL TABLES IN SCHEMA spdx TO chaoss; 
GRANT SELECT ON ALL TABLES IN SCHEMA augur_operations TO chaoss;
ALTER DEFAULT PRIVILEGES IN SCHEMA augur_data
GRANT SELECT ON TABLES TO chaoss;
```

## Augur Database Credentials
There are two directories the project starts with: 
1. `CHAOSS-Example`, which is an example against a publicly available Augur database of the CHAOSS Project's organization on GitHub and 
2. `templates`, which is a copy of the same notebooks found in `CHAOSS-Example` that we intend you to make a copy of for your project, which you can do on most linux based systems by running the command `cp -R templates my-project-name` (consider replacing `my-project-name` with a meaningful project name).
3. In your new directory, edit the `config.json` file in a text editor so that it contains credentials for your Augur database. 
In the directory where you want to run Jupyter Lab from, create a file called "config.json": 
```
{
    "connection_string": "sqlite:///:memory:",
    "database": "augur_mine",
    "host": "servernameorip.augurlabs.io",
    "password": "<your password>",
    "port": 5433,
    "schema": "augur_data",
    "user": "augur",
    "user_type": "read_only"
}
```

# Opening Reports in Jupyter Lab
1. From your augur-community-reports home directory, with your local python virtual environment activated, and requirements installed, run the command: `jupyter lab`. 

In any **NEW** jupyter notebooks, place this text as the first cell, if you do not begin by copying one of the existing notebooks: 
```
import psycopg2
import pandas as pd 
import sqlalchemy as salc
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import warnings
import datetime
import json
warnings.filterwarnings('ignore')

with open("config.json") as config_file:
    config = json.load(config_file)

database_connection_string = 'postgresql+psycopg2://{}:{}@{}:{}/{}'.format(config['user'], config['password'], config['host'], config['port'], config['database'])

dbschema='augur_data'
engine = salc.create_engine(
    database_connection_string,
    connect_args={'options': '-csearch_path={}'.format(dbschema)})

```

# Running New Contributor and Pull Request Templates
## Using the Control Cell
In both the pull request and contributor templates the control cell is used to configure what is in the report. 

### Variables in Both Templates
1. **Repo_set**: Takes a list of repo_ids you need visualizations for.
2. **Display_grouping**: Can be set as 'repo' or 'competitors'. 'repo' groups the visualizations by repo, and 'competitors' groups the visualizations by chart, so data can be easily compared against other repos.
3. **Not_alised_repos**: Takes a list of repo_ids you do not want aliased, when display_grouping is set to 'competitors'
4. **Save_files**: Can be set to True or False, when True all the visualizations will be export as PNG's
5. **Begin_date and End_date**: Take a string in date form, i.e. '2020-03-30'

### Variables for New Contributor Template
1. **Group_by**: Determines how data is grouped in bar charts. Can be set to 'year', 'quarter', or 'month'
2. **Time and Num_contributions_required**: Constraints for a repeat contributor. Indicate the number of contributions a contributor must make in the time to be considered a repeat contributor. Time is in days.

### Variables for Pull Request Template
1. **scatter_plot_outliers_removed**: Indicates the number of outliers you would like to remove on the days_to_first_response scatter plot. When you have a small number of outliers, this variable is useful for improving the utility of the visualizations. 



## License, Copyright, and Funding

Copyright © 2023 University of Nebraska at Omaha, University of Missouri, CHAOSS Project at the Linux Foundation, Sean Goggins

Notebooks in intial release Authored by Andrew Brain, Sean Goggins, Dawn Foster and Gabe Heim. Don't believe everything you see in a commit history. ;)

Augur is free software: you can redistribute it and/or modify it under the terms of the MIT License as published by the Open Source Initiative. See the [LICENSE](LICENSE) file for more details.

This work has been funded through the Alfred P. Sloan Foundation, Mozilla, The Reynolds Journalism Institute, and 9 Google Summer of Code Students. 

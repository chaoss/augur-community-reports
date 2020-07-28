# augur-community-reports
Building reports from the Augur database schema is an important way for cataloging questions that our users ask, as well as how we about using them. Over time, I expect these reports to evolve toward standard APIS in Augur. This is a repository for engaged requirements gathering in the agile spirit. 

The directories labeled "report" are for "reports" for one enterprise or project. The goal is to then build generalized augur queries/APIs, etc, and make them available as tools. 

The directories labeled "research" are for research paper related code that uses Augur. 

# Setup
## Prerequisites
1. Python 3.x
2. pip
3. virtualenv package `pip3 install virtualenv`

## Setup augur-community-reports
1. Fork the augur-community-reports repository
2. Clone your fork of the repository locally
```
git clone https://github.com/<your-fork>/augur-community-reports
````
3. Create your python virtual environment wherever you routinely store them. We use a `virtualenvs` directory. 
```
virtualenv --python=python3 virtualenvs/augur-community-reports
```
4. Activate your virtual environment
```
source  ../../virtualenvs/augur-community-reports/bin/activate
```
5. Install the necessary Python libraries
```
pip install -r requirements.txt
```
6. Change into the directory of your clone
```
cd augur-community-reports
```

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

database_connection_string = 'postgres+psycopg2://{}:{}@{}:{}/{}'.format(config['user'], config['password'], config['host'], config['port'], config['database'])

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



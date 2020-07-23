# community-reports
Building reports from the Augur database schema is an important way for catalogouing questions that our users ask, as well as how we about using them. Over time, I expect these reports to evolve toward standard APIS in Augur. This is a repository for engaged requirements gathering in the agile spirit. 

The directories labeled "report" are for "reports" for one enterprise or project. The goal is to then build generalized augur queries/APIs, etc, and make them available as tools. 

The directories labeled "research" are for research paper related code that uses Augur. 

# Setup
## Python Virtual Environment
```
virtualenv --python=python3 virtualenvs/augur-explorer
git clone https://github.com/augurlabs/augur-explorer 
cd augurlabs/augur-explorer
source activate ../../virtualenvs/augur-explorer/bin/activate
source  ../../virtualenvs/augur-explorer/bin/activate
pip install -r requirements.txt
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

In your jupyter notebooks, place this text as the first cell: 
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
Repo_set: Takes a list of repo_ids you need visualizations for.
Display_grouping: Can be set as 'repo' or 'competitors'. 'repo' groups the visualizations by repo, and 'competitors' groups the visualizations by chart, so data can be easily compared against other repos.
Not_alised_repos: Takes a list of repo_ids you do not want aliased, when display_grouping is set to 'competitors'
Save_files: Can be set to True or False, when True all the visualizations will be export as PNG's
Begin_date and End_date: Take a string in date form, i.e. '2020-03-30'

### Variables for New Contributor Template
Group_by: Determines how data is grouped in bar charts. Can be set to 'year', 'quarter', or 'month'
Time and Num_contributions_required: Constraints for a repeat contributor. Indicate the number of contributions a contributor must make in the time to be considered a repeat contributor. Time is in days.

### Variables for Pull Request Template
scatter_plot_outliers_removed: Indicates the number of outliers you would like to remove on the days_to_first_response scatter plot.



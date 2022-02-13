# Purpose and motivation

This repo is an extension of the [weather-etl](https://github.com/jonathanneo/weather-etl) example repo. The weather-etl example provides students of data engineering an example of an end-to-end ETL project that runs locally. 

This project, weather-etl-aws, provides student an example of the same project hosted on AWS. 


# Repo structure 
```
.github/workflows                           # contains continuous integration pipelines 
data/                                       # contains static datasets 
docs/                                       # contains additional documentation 
images/                                     # contains images used for the README
lambda_app/    
    |__ _config.template.sh                 # template for adding credentials and secrets 
    |__ ddl_create_table.sql                # SQL code used to create the target tables 
    |__ etl.py                              # an auto-generated file from the python ETL notebook (do not write your code here)
    |__ test_transformation_functions.py    # pytest unit tests 
    |__ transform_functions.py              # custom user-generated transformation functions 
    |__ requirements.txt                    # python dependencies for lambda app 
    |__ build.sh                            # shell script to build the zip file 
README.md                                   # all you need to know is in here 
```

# Solution 

## Solution architecture 

The solution architecture diagram was created using: https://draw.io/ 

Icons were taken from: https://www.flaticon.com/ and https://www.vecta.io/ 


![solution_architecture.drawio.png](images/solution_architecture.drawio.png)

# Running locally 

Follow the steps below to run the code locally: 

- [1. Declare environment variables](#declare-environment-variables)
- [2. Run the application locally ](#run-the-application-locally)


### Declare environment variables 

If you look into the `etl_lambda.py` file, you will see that there are several lines for: 

```python 
# getting CSV_ENDPOINT_CAPITAL_CITIES environment variable 
CSV_ENDPOINT_CAPITAL_CITIES = os.environ.get("CSV_ENDPOINT_CAPITAL_CITIES")

# getting API_KEY_OPEN_WEATHER environment variable 
API_KEY_OPEN_WEATHER = os.environ.get("API_KEY_OPEN_WEATHER")
```

These lines are used to store variables that are either (1) secrets, or (2) change between environments (e.g. dev, test, production). 

We will first need to declare the values for these variables. This can easily be done by running the following in the terminal: 

<b>macOS:</b> 
```
export CSV_ENDPOINT_CAPITAL_CITIES="path_to_csv"
export API_KEY_OPEN_WEATHER="secret_goes_here"
export DB_USER="secret_goes_here" # e.g. postgres 
export DB_PASSWORD="secret_goes_here" # e.g. postgres 
export DB_SERVER_NAME="secret_goes_here" # e.g. localhost
export DB_DATABASE_NAME="secret_goes_here"
```

<b>windows:</b> 

```
set CSV_ENDPOINT_CAPITAL_CITIES="path_to_csv"
set API_KEY_OPEN_WEATHER="secret_goes_here"
set DB_USER="secret_goes_here" # e.g. postgres 
set DB_PASSWORD="secret_goes_here" # e.g. postgres 
set DB_SERVER_NAME="secret_goes_here" # e.g. localhost
set DB_DATABASE_NAME="secret_goes_here"
```

To save time running each variable in the terminal, you may wish to create script files to store the declaration of each variable. 

- macOS: store the declaration of the variables in a `config.local.sh` file 
    - run using `. ./config.local.sh` 
- windows: store the declaration of the variables in a `config.local.bat` file 
    - run using `. ./config.local.bat` 


### Run the application locally 

To run the application locally, simply run `etl_lambda_local.py`. 

`etl_lambda_local.py` will import the main function in `etl_lambda.py` and run it. 

If successful, you should see records appearing in your local postgres database. 

# AWS deployment  

Follow these steps to deploy the solution to AWS. 

- 

## Python dependencies 
The required python libraries and version have been specified in [requirements.txt](requirements.txt). 

Install python dependencies by performing : 

```
pip install -r requirements.txt 
```

## Credentials 
In the `script/` folder, create a `credentials.py` file with the following variables:
```py
api_key = "<your_api_key>"                  # open weather API api key 
db_user = "<your_database_user>"            # postgresql username 
db_password = "<your_database_password>"    # postgresql password 
```

These credentials will be used in the `etl.ipynb` notebook. 

The `credentials.py` file is already in .gitignore and thus your credentials will not be stored on Git. 

## Running code locally 
To run the ETL code on your computer, execute the following in your terminal: 

```
cd scripts
python -m jupyter nbconvert --to python scripts/etl.ipynb
python scripts/etl.py
```

## Run unit tests 
To run the unit tests on your computer, execute the following in your terminal: 

```
pytest scripts
```

You should see the following output: 

```
====== test session starts ======
platform darwin -- Python 3.7.11, pytest-6.2.5, py-1.11.0, pluggy-1.0.0
collected 2 items
scripts/test_transformation_functions.py .. [100%]
====== 2 passed in 0.36s ======
```

## Continuous integration 

To ensure that code is tested prior to merging to the `main` branch, an automated Continuous Integration (CI) pipeline has been configured. 

See code [here](.github/workflows/etl-ci.yml). 

The expected output when the CI pipeline runs are: 

1. Successful execution of CI pipeline 

![ci-pipeline.png](images/ci-pipeline.png)


2. All unit tests passed 

![ci-test-output.png](images/ci-test-output.png)



## Scheduling jobs 


<details>
<summary><strong> Cron (MacOS or Linux) </strong></summary>

To schedule a job using cron (see full guide [here](https://ole.michelsen.dk/blog/schedule-jobs-with-crontab-on-mac-osx/)): 

```sh 
# open crontab 
env EDITOR=nano crontab -e
# paste the following into crontab
* * * * * cd /Users/jonathanneo/Documents/trilogy/weather-etl/scripts && bash run_etl.sh
# write the file using: CTRL + O
# close the file using: CTRL + X
# check that the cron job has been scheduled - you should see your job appear 
crontab -l 
```

You should see the following output: 
```
(base) Jonathans-MacBook-Pro-2:~ jonathanneo$ crontab -l
* * * * * cd /Users/jonathanneo/Documents/trilogy/weather-etl/scripts && bash run_etl.sh
```

</details>

<details>
<summary><strong> Task Scheduler (Windows) </strong></summary>

1. Open Task Scheduler on windows 

2. Select `Create task`

![images/task-scheduler-1.png](images/task-scheduler-1.png)

3. Provide a name for the task 

![images/task-scheduler-2.png](images/task-scheduler-2.png)

4. Select `Actions` > `New` 

![images/task-scheduler-3.png](images/task-scheduler-3.png)

5. Provide the following details, and click `OK`: 
    - Program/script: `<provide path to your python.exe in your conda environment folder>`
        - Example: `C:\Users\jonat\anaconda3\envs\PythonData\python.exe`
    - Add arguments (optional): `<provide the etl file>`
        - Example: `etl.py` 
    - Start in (optional): `<provide the path to the etl file>` 
        - Example: `C:\Users\jonat\Documents\weather-etl\scripts`

![images/task-scheduler-4.png](images/task-scheduler-4.png)

6. Select `Triggers` 

![images/task-scheduler-5.png](images/task-scheduler-5.png)

7. Provide details of when you would like the job to run 

![images/task-scheduler-6.png](images/task-scheduler-6.png)

8. Click `OK` 

</details>

# Contributors
- [@jonathanneo](https://github.com/jonathanneo)
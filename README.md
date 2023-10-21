# apache-superset-deploy
This demonatration was for one of my respective client. I have documented it for everyone who needs.
Apache Superset™ is an open-source modern data exploration and visualization platform.
You can get details from below link.
https://superset.apache.org/docs/intro

For production use it needs different type of software to setup, config and connect, but we are going to do a development setup.
There are lot of option to do it(docker, pypi or others) but we are going to start from sratch.
https://superset.apache.org/docs/installation/installing-superset-from-scratch/

## We are going to use an Ubuntu 23.04 vm for this installation.
1. Run below command to install necessary software packages.
```
apt install build-essential libssl-dev libffi-dev python3-dev python3-pip libsasl2-dev libldap2-dev default-libmysqlclient-dev
    apt install mysql-server
    mysql_secure_installation
```
> (set the config as per your need)
#login to mysql create a user, db and set the user permission to access the db:
mysql> create user 'superset'@'localhost' identified by 'password';
Query OK, 0 rows affected (0.03 sec)

mysql> grant all on `rnd_superset%`.* to 'superset'@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> create database rnd_superset;
Query OK, 1 row affected (0.00 sec)

#Create a second db for your test data
mysql> create database rnd_superset_data;
Query OK, 1 row affected (0.00 sec)

3. Let's also make sure we have the latest version of pip and setuptools:
pip install --upgrade setuptools pip

4. Install python virtual environment
pip install virtualenv

5. Lets asume you are logged in as root and you are in the root directory, you can check your current directory by pwd command.
pwd
mkdir -p ~/superset
cd ~/superset
python3 -m venv venv
. venv/bin/activate

6. Installling superset in the virtual environment:
pip install apache-superset

6.#After installing superset we need to do some other config to run our app:
Reference: https://superset.apache.org/docs/installation/configuring-superset/
mkdir -p ~/superset/venv/app
nano ~/superset/venv/app/superset_config.py (put below config in the file and save it)
#We need to create a strong key to define for the SECRET_KEY otherwise you will get messege that superset is using default secret and superset will not run. To generate a string ke you can use:

openssl rand -base64 42

..............................superset_config.py.................................
`#Superset specific config
ROW_LIMIT = 5000

#Flask App Builder configuration
#Your App secret key will be used for securely signing the session cookie
#and encrypting sensitive information on the database
#Make sure you are changing this key for your deployment with a strong key.
#Alternatively you can set it with `SUPERSET_SECRET_KEY` environment variable.
#You MUST set this for production environments or the server will not refuse
#to start and you will see an error in the logs accordingly.
SECRET_KEY = '78dkudoZ9wk+CD6aNBfuM5QaLtJSMv4o5M+Ht8hBjA1HuDlzR9FBmQ1W'

#The SQLAlchemy connection string to your database backend
#This connection defines the path to the database that stores your
#superset metadata (slices, connections, tables, dashboards, ...).
#Note that the connection information to connect to the datasources
#you want to explore are managed directly in the web UI
#The check_same_thread=false property ensures the sqlite client does not attempt
#to enforce single-threaded access, which may be problematic in some edge cases
#SQLALCHEMY_DATABASE_URI = 'sqlite:////path/to/superset.db?check_same_thread=false'
SQLALCHEMY_DATABASE_URI = 'mysql://username:password@localhost/dbname'

#Flask-WTF flag for CSRF
WTF_CSRF_ENABLED = False
#Add endpoints that need to be exempt from CSRF protection
WTF_CSRF_EXEMPT_LIST = []
#A CSRF token that expires in 1 year
WTF_CSRF_TIME_LIMIT = 60 * 60 * 24 * 365

#Set this API key to enable Mapbox visualizations
MAPBOX_API_KEY = ''
#Set this otherwise you will face login issue after run in development mode.
TALISMAN_ENABLED = False`
.........................superset_config.py....................................

7. Initialize the database and Create an admin user in your metadata database (use `admin` as username to be able to load the examples)
export FLASK_APP=superset
superset db upgrade
#Note at this point you can se some error that mysqlclient not found or something, you need to install it if so:
apt install pkg-config #run this in other terminal, not in your virtualenv
pip install mysqlclient #run in your virtualenv
#if you found more difficulties to install it you can check below link for any clue/solution
https://stackoverflow.com/questions/76585758/mysqlclient-cannot-install-via-pip-cannot-find-pkg-config-name#:~:text=The%20error%20you%20are%20encountering,build%20the%20mysqlclient%20Python%20package.
#add a username in the input field, First name, lastname, email and password)
superset fab create-admin
#Load some data to play with
superset load_examples
#Create default roles and permissions
superset init

8. To start a development web server on port 8088, use -p to bind to another port and use --host to allow from all or specific IP
superset run -p 8088 --with-threads --reload --debugger #it will run on localhost only
superset run -p 8088 --host=0.0.0.0 --with-threads --reload --debugger #it will allow all to browse use http://IP:PORT

9. Lets login and explore. You can add your test data from the top right corner:
Settings > Database Connections > DATABASE > MySQL > add connection details and test connection
Note: You already have example data loaded during insatllation, it is stored in sqlite db by default. You can download example data also and import those in your mysql db.

Thanks, hope this gude will help to understand the apache superset setup.







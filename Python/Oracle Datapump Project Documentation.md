#python #oracle #rds 
## Project Overview

This project is a Streamlit web application designed for efficient management of Oracle databases, specifically focusing on database export and import using Oracle Data Pump. The application provides a user-friendly interface for managing connections, querying databases, and executing Data Pump operations.

## Technologies Used

- **Python**: Core programming language.
- **Streamlit (v1.42.2)**: Web application framework for a clean, interactive UI.
- **Oracle Database (oracledb v1.4.2)**: Database driver for connecting to Oracle DB.
- **Pandas (v2.1.3)**: Data manipulation and visualization within the app.
- **Docker**: Containerization using a custom Dockerfile.

## Project Structure

- **app.py**: The main application file containing all Streamlit UI components and functionality.
- **requirements.txt**: List of required Python packages.
- **connections.csv**: CSV file storing database connection strings.
- **Dockerfile**: Docker configuration file for containerizing the application.
- **SQL Files**:
    
    - `constraints.sql`: SQL script for managing database constraints.
    - `datapumpdir.sql`: SQL script for managing Data Pump directories.
    - `dbaobjects.sql`: SQL script for fetching database object information.
    - `movedmp.sql`: SQL script for moving Data Pump files.
    - `rdsdisk.sql`: SQL script for checking RDS disk space.
    - `recompile.sql`: SQL script for recompiling database objects.
    - `schemaddl.sql`: SQL script for generating schema DDL.
        
## Main Python Code (app.py)

```python
import streamlit as st
import oracledb
from subprocess import run, PIPE

oracledb.init_oracle_client(lib_dir="/Users/marcospozzetti/Documents/instantclient_23_3")

st.set_page_config(layout='wide')

# Get Strings or SID from all.csv file
def getStrings(file, sid=None, type=int):
    with open(file, 'r') as f:
        if type == 3:
            return [line.split(',')[0] for line in f]
        else:
            for line in f:
                line = line.split(',')
                if line[0] == sid and type == 0:
                    return line[1].strip()
                elif line[0] == sid and type == 1:
                    return line[1].strip()
                elif line [0] == sid and type == 2:
                    return line[2].strip()

movedmp = open('./sql/movedmp.sql', 'r').read()
connections = './utils/connections.csv'
iaasdirs = open('./sql/iaasdirs.sql', 'r').read()
conn = (getStrings(connections, None, 3))

# Standard function to query the DB
def queryDB(db, query):
    try:
        connection = oracledb.connect(getStrings(connections, db, 1))
        cursor = connection.cursor()
        cursor.execute(query)
        return cursor.fetchall()
    except Exception as e:
        st.error(f'Error querying database: {e}')
    finally:
        connection.commit()
        connection.close()

def expdp():
    st.title('Export')
    source = st.selectbox('Select a source database:', conn)
    result = queryDB(source, iaasdirs)
    processed_data = [item[0].strip().split(',', 1)[0] for item in result]
    dir = st.selectbox('DATA PUMP DIR', processed_data)
    dumpfile = st.text_input('DUMPFILE NAME')
    parfile  = st.text_input('PARFILE NAME')
    logfile = st.text_input('LOGFILE NAME')
    query = queryDB(source, 'select username from dba_users')
    processed_data = [item[0].strip().split(',', 1)[0] for item in query]
    schemas = st.multiselect('SCHEMAS', processed_data)
    schemas = ','.join(schemas)
    tables = st.text_input('TABLES (leave in blank if not needed)')
    folder = st.text_input('FOLDER (include the last /)')
    full = st.selectbox('FULL EXPORT', ['No', 'Yes'])
    createpar = st.button('CREATE PARFILE')
    button = st.button('EXECUTE')
    monitor = st.button('MONITOR FILE')
    if createpar:
        tnsname = getStrings(connections, source, 2)
        fol = st.write(run(f'mkdir {folder}', shell=True, capture_output=True, text=True))
        parfile_contents = []
        parfile_contents.append(f"USERID='expimpy/Davide123!@{tnsname}'")
        if dir: parfile_contents.append(f"directory={dir}")
        if dumpfile: parfile_contents.append(f"dumpfile={dumpfile}.dmp")
        if logfile: parfile_contents.append(f"logfile={logfile}.log")
        if schemas: parfile_contents.append(f"schemas={schemas}")
        if tables: parfile_contents.append(f"tables={tables}")
        if full == 'Yes': parfile_contents.append("FULL=Y")
        parfile_contents.append(f"METRICS=Y")
        parfile1 = "\n".join(parfile_contents)
        par_cmd = run(f'echo "{parfile1}" > {folder}{parfile}.par', shell=True, capture_output=True, text=True)
        st.write(f'Parameter file creation output: {par_cmd.stdout}')
        output = st.write(run(f'cat {folder}{parfile}.par', shell=True, capture_output=True, text=True))
    if button:
        exec_cmd = st.write(run(f'expdp parfile={folder}{parfile}.par >> {folder}{logfile}.log', shell=True, capture_output=True, text=True))
        st.write(f'Export operation output: {exec_cmd.stdout}')
        st.write(f'Export errors if any: {exec_cmd.stderr}')
    if monitor:
        exec_cmd = st.write(run(f'tail -f {folder}{logfile}.par', shell=True, capture_output=True, text=True))

def dmpmove():
    st.title('Move Dumpfile from Server to RDS')
    source = st.selectbox('Select a source database:', conn)
    result = queryDB(source, iaasdirs)
    processed_data = [item[0].strip().split(',', 1)[0] for item in result]
    dir = st.selectbox('Source Data Pump Dir', processed_data)
    targetdir = st.selectbox('Target Data Pump Dir', processed_data)
    target = st.selectbox('Select a target database:', conn)
    dumpfile = st.text_input('DUMPFILE NAME')
    move = st.button('Move File')
    if move:
        st.write(movedmp)
        query = movedmp.replace('PY-DIR', dir)\
               .replace('PY-DUMP', dumpfile)\
               .replace('PY-TARGET-DIR', targetdir)\
               .replace('PY-DBLINK', target)
        st.write(query)
        queryDB(source, query)

def impdp():
    st.title('Import')
    target = st.selectbox('Select a target database:', conn)
    result = queryDB(target , iaasdirs)
    processed_data = [item[0].strip().split(',', 1)[0] for item in result]
    dir = st.selectbox('DATA PUMP DIR', processed_data)
    dumpfile = st.text_input('DUMPFILE NAME')
    parfile  = st.text_input('PARFILE NAME')
    logfile = st.text_input('LOGFILE NAME')
    query = queryDB(target, 'select username from dba_users')
    processed_data = [item[0].strip().split(',', 1)[0] for item in query]
    tables = st.text_input('TABLES (leave in blank if not needed)')
    folder = st.text_input('FOLDER (include the last /)')
    full = st.selectbox('FULL EXPORT', ['No', 'Yes'])
    createpar = st.button('CREATE PARFILE')
    button = st.button('EXECUTE')
    monitor = st.button('MONITOR FILE')
    if createpar:
        tnsname = getStrings(connections, target, 2)
        fol = st.write(run(f'mkdir {folder}', shell=True, capture_output=True, text=True))
        parfile_contents = []
        parfile_contents.append(f"USERID='expimpy/Davide123!@{tnsname}'")
        if dir: parfile_contents.append(f"directory={dir}")
        if dumpfile: parfile_contents.append(f"dumpfile={dumpfile}.dmp")
        if logfile: parfile_contents.append(f"logfile={logfile}.log")
        if tables: parfile_contents.append(f"tables={tables}")
        parfile_contents.append(f"METRICS=Y")
        parfile1 = "\n".join(parfile_contents)
        par_cmd = run(f'echo "{parfile1}" > {folder}{parfile}.par', shell=True, capture_output=True, text=True)
        st.write(f'Parameter file creation output: {par_cmd.stdout}')
        output = st.write(run(f'cat {folder}{parfile}.par', shell=True, capture_output=True, text=True))
    if button:
        exec_cmd = st.write(run(f'impdp parfile={folder}{parfile}.par >> {logfile}.log', shell=True, capture_output=True, text=True))
        st.write(f'Export operation output: {exec_cmd.stdout}')
        st.write(f'Export errors if any: {exec_cmd.stderr}')
    if monitor:
        exec_cmd = st.write(run(f'tail -f {folder}{logfile}.par', shell=True, capture_output=True, text=True))

page = st.sidebar.radio("Navigation", ['Export', 'Move Dumpfile', 'Import'])

if page == 'Export':
    expdp()
elif page == 'Move Dumpfile':
    dmpmove()  
elif page == 'Import':
    impdp()
```

## Features

- **Add Database Connection**: Allows users to add new Oracle DB connections.
    
- **Disk & Free Space Monitoring**: Retrieves and displays available disk space for a selected database.
    
- **Data Pump Directories Management**: Lists available Data Pump directories in source and target databases.
    
- **Database Export (EXPDP)**: User-friendly interface to generate and execute Data Pump export.
    
- **Database Import (IMPDP)**: Easy setup for importing Data Pump files.
    
- **Schema Management**: Generates and applies schema DDL based on user selection.
    
- **Object Management**: Displays database object details (tables, indexes, etc.).
    
- **Recompile Objects**: Recompiles all invalid objects in a selected database.
    

## Docker Setup

1. Build the Docker image:

```
docker build -t oracle-datapump-app .
```

2. Run the container:

```
docker run -p 8501:8501 oracle-datapump-app
```

## Usage

### Setup

1. Ensure Python is installed.
2. Install the required packages using:
    
```
pip install -r requirements.txt
```

3. Make sure to configure Oracle Instant Client with the path in `app.py`.
### Running the Application

```
streamlit run app.py
```

### Adding a Database Connection

- Navigate to the **'Add Connection'** page.
    
- Provide database name, host, port, service name, username, and password.
    
- Click **'Add Connection'** to save the connection string in `connections.csv`.
    
### Exporting a Database

- Go to the **'Export'** section.
    
- Choose the source database, Data Pump directory, and set file names (dump, parfile, logfile).
    
- Select schemas and tables if needed.
    
- Click **'EXECUTE'** to start the export.
    
### Importing a Database

- Go to the **'Import'** section.
    
- Choose the target database and Data Pump directory.
    
- Specify dumpfile, parfile, and logfile names.
    
- Click **'EXECUTE'** to start the import.

### Schema Management

- Go to the **'Create Schema and Roles'** page.
    
- Select source and target databases.
    
- Choose schemas to export DDL.
    
### Object Management

- Go to the **'Objects'** page.
    
- Select source or target database.
    
- View object details like tables, indexes, triggers, etc.
    
### Recompile Objects

- Navigate to the **'Recompile'** page.
    
- Select the target database.
    
- Click **'Execute'** to recompile all invalid objects.
    
## Best Practices

- Regularly update `connections.csv` to keep database credentials up to date.
    
- Use strong, unique passwords for database connections.
    
- Monitor export and import logs for errors or warnings.
    
## Security

- Ensure `connections.csv`, Dockerfile, and all SQL scripts are securely stored.
    
- Restrict access to the application to authorized users only.
#bash #oracle 

```bash
#!/bin/bash

# Define the connection strings with explicit host, port, and SID
PROD="master/password@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=flooratestdump.cqhnw9qw0vvk.eu-west-1.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=FLOORA)))"
SVIL="master/password@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=floorasvil19.cqhnw9qw0vvk.eu-west-1.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=FLOORA)))"

# Get the directory of the script
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Create the logs directory if it doesn't exist
LOG_DIR="$SCRIPT_DIR/logs"
mkdir -p "$LOG_DIR"

# Generate a log file name with the current date and time
LOG_FILE="$LOG_DIR/session_kill_$(date '+%Y%m%d_%H%M%S').log"

# Prompt the user for the connection string
echo "Which connection string would you like to use? (prod/svil)"
read -r CONNECTION_CHOICE

# Set the connection string based on user input
if [[ "$CONNECTION_CHOICE" == "prod" ]]; then
  CONNECTION_STRING=$PROD
elif [[ "$CONNECTION_CHOICE" == "svil" ]]; then
  CONNECTION_STRING=$SVIL
else
  echo "Invalid choice. Please enter 'prod' or 'svil'."
  exit 1
fi

# Prompt the user for the username
echo "Enter the username for the schema:"
read -r USERNAME

# Find the first .env file in the user's home directory
FIRST_ENV_FILE=$(find "$HOME" -type f -name "*.env" | head -n 1)

if [[ -n "$FIRST_ENV_FILE" ]]; then
  ENV_FILE=$(basename "$FIRST_ENV_FILE")
  echo "The first .env file found is: $ENV_FILE"
  
  # Source the environment file
  source "$FIRST_ENV_FILE"
else
  echo "No .env file found in $HOME"
fi

# Log the sessions before attempting to kill them
sqlplus -s "$CONNECTION_STRING" <<EOF | tee -a "$LOG_FILE"
SET PAGESIZE 50000;
SET LINESIZE 200;
SET FEEDBACK OFF;
SET HEADING ON;

SPOOL $LOG_FILE APPEND

SELECT sid, serial#, username, module
FROM v\$session
WHERE username = '$USERNAME';

SPOOL OFF
EOF

# Connect to the Oracle server and execute the SQL block to kill sessions
sqlplus -s "$CONNECTION_STRING" <<EOF | tee -a "$LOG_FILE"
SET SERVEROUTPUT ON;

DECLARE
  v_killed_sessions NUMBER := 0;
BEGIN
  FOR session IN (
    SELECT sid, serial#
    FROM v\$session
    WHERE username = '$USERNAME'
  )
  LOOP
    BEGIN
      rdsadmin.rdsadmin_util.kill(
        sid => session.sid,
        serial => session.serial#,
        method => 'IMMEDIATE'
      );
      v_killed_sessions := v_killed_sessions + 1;
    EXCEPTION
      WHEN OTHERS THEN
        NULL; -- Handle exceptions if needed
    END;
  END LOOP;
  
  DBMS_OUTPUT.PUT_LINE('Total sessions killed: ' || v_killed_sessions);
END;
/
EOF
```
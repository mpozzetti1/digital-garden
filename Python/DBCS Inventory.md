#python #dbcs

```python
import oci
import csv
from datetime import datetime

# Configure the OCI Python SDK
config = oci.config.from_file(file_location="~/.oci/config", profile_name="DEFAULT")
database_client = oci.database.DatabaseClient(config)
identity_client = oci.identity.IdentityClient(config)

# Get the tenancy ID
tenancy_id = config["tenancy"]

# Get all compartments
compartments = oci.pagination.list_call_get_all_results(
    identity_client.list_compartments,
    tenancy_id,
    compartment_id_in_subtree=True
).data

# Open a CSV file for writing
csv_file = open("dbs_inventory.csv", "w", newline="")
csv_writer = csv.writer(csv_file)

# Write the CSV header
csv_writer.writerow(["Compartment Name", "Compartment ID", "DBS Name", "DBS ID", "Shape", "Edition", "CPU Core Count", "Node Count", "Storage Size (GB)", "License Model", "DB Version", "DB Workload", "Created"])

# Iterate through compartments and list DBSs
for compartment in compartments:
    try:
        dbs_list = oci.pagination.list_call_get_all_results(
            database_client.list_db_systems,
            compartment_id=compartment.id
        ).data

        for dbs in dbs_list:
            csv_writer.writerow([
                compartment.name,
                compartment.id,
                dbs.display_name,
                dbs.id,
                dbs.shape,
                dbs.database_edition,
                dbs.cpu_core_count,
                dbs.node_count,
                dbs.data_storage_size_in_gbs,
                dbs.license_model,
                dbs._version,  # Changed from dbs.db_version
            ])
    except oci.exceptions.ServiceError as e:
        print(f"Error: {e}")

# Close the CSV file
csv_file.close()
```
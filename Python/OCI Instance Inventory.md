#python #instances

```python
import oci
import csv
import subprocess
from datetime import datetime, timedelta
import difflib
import matplotlib.pyplot as plt
import os
import matplotlib.cm as cm
import numpy as np
import oci.pagination
import oci.exceptions
from oci.monitoring.models import ListMetricsDetails

config = oci.config.from_file(file_location="~/.oci/config", profile_name="DEFAULT")
identity_client = oci.identity.IdentityClient(config)
user = identity_client.get_user(config["user"]).data
tenancy_id = config["tenancy"]
region = config["region"]

compute_client = oci.core.ComputeClient(config)
network_client = oci.core.VirtualNetworkClient(config)
monitoring_client = oci.monitoring.MonitoringClient(config)

def get_instance_ip_addresses(compute_client, virtual_network_client, instance):
    public_ips = []
    private_ips = []

    vnic_attachments = oci.pagination.list_call_get_all_results(
        compute_client.list_vnic_attachments,
        compartment_id=instance.compartment_id,
        instance_id=instance.id
    ).data

    vnics = [virtual_network_client.get_vnic(va.vnic_id).data for va in vnic_attachments]
    for vnic in vnics:
        if vnic.public_ip:
            public_ips.append(vnic.public_ip)

        private_ips_for_vnic = oci.pagination.list_call_get_all_results(
            virtual_network_client.list_private_ips,
            vnic_id=vnic.id
        ).data

        for private_ip in private_ips_for_vnic:
            private_ips.append(private_ip.ip_address)

            try:
                public_ip = virtual_network_client.get_public_ip_by_private_ip_id(
                    oci.core.models.GetPublicIpByPrivateIpIdDetails(
                        private_ip_id=private_ip.id
                    )
                ).data

                if public_ip.ip_address not in public_ips:
                    public_ips.append(public_ip.ip_address)
            except oci.exceptions.ServiceError as e:
                if e.status != 404:
                    print(f'Unexpected error when retrieving public IPs: {str(e)}')

    return public_ips, private_ips

compartments = oci.pagination.list_call_get_all_results(
    identity_client.list_compartments,
    tenancy_id,
    compartment_id_in_subtree=True
).data

column_names = ['Hostname', 'Link', 'State', 'Type', 'ID', 'Public IP', 'Private IP', 'Average CPU Usage (30 days)', 'Compartment']

Hostname = []
Link = []
State = []
Type = []
Id = []
PublicIp = []
PrivateIp = []
CpuUsage = []
Compartment = []

for compartment in compartments:
    compartment_id = compartment.id

    instances = oci.pagination.list_call_get_all_results(
        compute_client.list_instances,
        compartment_id=compartment_id,
        sort_by="TIMECREATED",
    ).data

    for instance in instances:
        State.append(instance.lifecycle_state)
        Type.append(instance.shape)
        Id.append(instance.id)
        Link.append(f"https://cloud.oracle.com/compute/instances/ocid1.instance.oc1.{region}.{instance.id}?region={region}")

        os_value = instance.metadata.get('Operating System', '')
        hostname_value = instance.display_name
        Hostname.append(hostname_value)
        Compartment.append(compartment_id)

        public_ips, private_ips = get_instance_ip_addresses(compute_client, network_client, instance)
        PublicIp.append(public_ips[0] if public_ips else None)
        PrivateIp.append(private_ips[0] if private_ips else None)

cpu_usage = None
try:
    list_metrics_details = ListMetricsDetails()
    list_metrics_details.compartment_id = compartment_id
    list_metrics_details.namespace = "oci_computeagent"
    list_metrics_details.resource_group = "instance"
    list_metrics_details.name = "CpuUtilization"
    list_metrics_details.resource_ids = [instance.id]
    list_metrics_details.start_time = datetime.now() - timedelta(days=30)
    list_metrics_details.end_time = datetime.now()
    list_metrics_details.resolution = "1m"
    list_metrics_details.period = "30d"

    cpu_metrics = monitoring_client.list_metrics(compartment_id, list_metrics_details=list_metrics_details).data

    if cpu_metrics:
        cpu_usage = cpu_metrics[0].data_points[-1].value
        cpu_usage = f"{cpu_usage:.2f}%"
except oci.exceptions.ServiceError as e:
    print(f"Error retrieving CPU usage for instance {instance.id}: {e}")

CpuUsage.append(cpu_usage)

rows = []
for i in range(len(Hostname)):
    public_ip = PublicIp[i] if i < len(PublicIp) else None
    private_ip = PrivateIp[i] if i < len(PrivateIp) else None
    cpu_usage = CpuUsage[i] if i < len(CpuUsage) else None
    row = [Hostname[i], Link[i], State[i], Type[i], Id[i], public_ip, private_ip, cpu_usage, Compartment[i]]
    rows.append(row)

today = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
csv_filename = f"instance_data_{today}.csv"
bck_folder = "BCK"
images_folder = f"{bck_folder}/images"

if not os.path.exists(bck_folder):
    os.makedirs(bck_folder)
if not os.path.exists(images_folder):
    os.makedirs(images_folder)

bck_file_path = f"{bck_folder}/{csv_filename}"
with open(bck_file_path, 'w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(column_names)
    writer.writerows(rows)

print(f"CSV file '{bck_file_path}' has been created.")

instance_types = [row[3] for row in rows]
instance_states = [row[2] for row in rows]

plt.figure(figsize=(10, 6))
instance_types_unique = list(set(instance_types))
colors = cm.rainbow(np.linspace(0, 1, len(instance_types_unique)))
bars = plt.barh(range(len(instance_types_unique)), [instance_types.count(t) for t in instance_types_unique], color=colors)
plt.yticks(range(len(instance_types_unique)), instance_types_unique)
plt.xlabel('Count')
plt.ylabel('Instance Type')
plt.title('Instance Types Distribution')
plt.tight_layout()
plot1_path = f"{images_folder}/plot1_{today}.png"
plt.savefig(plot1_path)
plt.close()

plt.figure(figsize=(10, 6))
instance_states_unique = list(set(instance_states))
colors = cm.rainbow(np.linspace(0, 1, len(instance_states_unique)))
bars = plt.barh(range(len(instance_states_unique)), [instance_states.count(t) for t in instance_states_unique], color=colors)
plt.yticks(range(len(instance_states_unique)), instance_states_unique)
plt.xlabel('Count')
plt.ylabel('Instance State')
plt.title('Instance States Distribution')
plt.tight_layout()
plot3_path = f"{images_folder}/plot3_{today}.png"
plt.savefig(plot3_path)
plt.close()

bck_files = sorted(os.listdir(bck_folder))
if len(bck_files) > 1:
    last_file = f"{bck_folder}/{bck_files[-2]}"
    new_file = bck_file_path

    if os.path.isfile(last_file) and os.path.isfile(new_file):
        with open(last_file, 'r') as f1, open(new_file, 'r') as f2:
            last_lines = f1.readlines()
            new_lines = f2.readlines()

        diff = difflib.unified_diff(last_lines, new_lines, fromfile=last_file, tofile=new_file)
        changes = []
        for line in diff:
            if line.startswith('+') and ',' in line:
                parts = line[1:].strip().split(',')
                hostname = parts[1]
                state = parts[3]
                if state == 'RUNNING':
                    changes.append(f"Instance '{hostname}' has been started.")
                elif state == 'STOPPED':
                    changes.append(f"Instance '{hostname}' has been stopped.")
            elif line.startswith('-') and ',' in line:
                parts = line[1:].strip().split(',')
                hostname = parts[1]
                changes.append(f"Instance '{hostname}' has been removed.")

        if changes:
            email_subject = "OCI Instance Data Changes"
            email_body = "FALCO OCI / Please find the OCI instance data changes below:\n\n" + "\n".join(changes)
            email_recipient = "marcos.pozzetti@veolia.com"

            # Send email with plots and CSV file
            email_command = f"echo '{email_body}' | mutt -s '{email_subject}' -a '{bck_file_path}' -a '{plot1_path}' -a '{plot3_path}' -- {email_recipient}"
            subprocess.run(email_command, shell=True)

            print("Email with instance changes, plots, and CSV file sent successfully.")
        else:
            print("No changes detected in instance hostnames or running/stopped state.")
    else:
        print("No previous file found in the BCK folder for comparison.")
else:
    print("No previous file found in the BCK folder for comparison.")

email_subject = "OCI Instance Data"
email_body = "FALCO OCI / Please find the OCI instance data attached."
email_recipient = "marcos.pozzetti@veolia.com"

email_command = f"echo '{email_body}' | mutt -s '{email_subject}' -a '{bck_file_path}' -a '{plot1_path}' -a '{plot3_path}' -- {email_recipient}"

subprocess.run(email_command, shell=True)

print("Email with CSV file sent successfully.")
```
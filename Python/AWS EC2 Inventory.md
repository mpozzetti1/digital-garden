#python #ec2 

```python
import boto3
import csv
import subprocess
from datetime import datetime, timedelta
from concurrent.futures import ThreadPoolExecutor

today = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
filename = (f'inventory{today}.csv')

ec2 = boto3.resource('ec2', region_name='eu-west-1')
cw = boto3.client('cloudwatch', region_name='eu-west-1')

instances = ec2.instances.all()

column_names = ['Hostname', 'Vpc', 'Type', 'State', 'OS', 'Public IP', 'Private IP', 'Average CPU Usage (30 days)', 'CPU Credit Balance', 'CPU Credit Usage', 'Average Ram Usage (30 days)', 'EBS Total GBs', 'EBS Used %', 'Backup', 'IAM Role', 'Private Key', 'ID', 'Link']

cpuMetric = 'CPUUtilization'
ramMetric = 'mem_used_percent'
diskMetric = 'disk_used_percent'
creditsMetric = 'CPUCreditBalance'
creditsUsage = 'CPUCreditUsage'

def getAvg(instance_id, namespace, start_time, end_time, metric, unit):
    response = cw.get_metric_statistics(
        Namespace=namespace,
        MetricName=metric,
        Dimensions=[
            {
                'Name': 'InstanceId',
                'Value': instance_id
            },
        ],
        StartTime=start_time,
        EndTime=end_time,
        Period=3600,
        Statistics=['Average'],
        Unit=unit
    )

    datapoints = response['Datapoints']
    if datapoints:
        total_usage = sum(datapoint['Average'] for datapoint in datapoints)
        average_usage = total_usage / len(datapoints)
        return average_usage
    else:
        return 0

def mainFunction(instance):
    State = instance.state['Name']
    Id = instance.id
    PublicIp = instance.public_ip_address
    PrivateIp = instance.private_ip_address
    Vpc = instance.vpc_id
    Link = f"https://eu-west-1.console.aws.amazon.com/ec2/home?region=eu-west-1#InstanceDetails:instanceId={Id}"
    os_value = ''
    hostname_value = ''
    backup_value = ''
    for tag in instance.tags:
        if tag['Key'] == 'OS':
            os_value = tag['Value']
        elif tag['Key'] == 'Name':
            hostname_value = tag['Value']
        elif tag['Key'] == 'Backup':
            backup_value = tag['Value']
        elif tag['Key'] == 'Name':
            hostname_value = tag['Value']
    Os = os_value
    Backup = backup_value
    iam_role = str(instance.iam_instance_profile).split('/')[-1].split(',')[0].strip("'")
    private_key = instance.key_name
    namespace1 = 'AWS/EC2'
    namespace2 = 'CWAgent'
    end = datetime.now()
    start = end - timedelta(days=30)
    ebs_total = sum(volume.size for volume in instance.volumes.all())
    cpu_avg = round(getAvg(Id, namespace1, start, end, cpuMetric, 'Percent'), 2)
    credits_avg = round(getAvg(Id, namespace1, start, end, creditsMetric, 'Count'), 2)
    credits_usage = round(getAvg(Id, namespace1, start, end, creditsUsage, 'Count'), 2)
    ram_avg = round(getAvg(Id, namespace2, start, end, ramMetric, 'Percent'), 2)
    disk_avg = round(getAvg(Id, namespace2, start, end, diskMetric, 'Percent'), 2)
    print(hostname_value)
    return [hostname_value, Vpc, instance.instance_type, State, Os, PublicIp, PrivateIp, str(cpu_avg) + '%', credits_avg, credits_usage, str(ram_avg) + '%', ebs_total, str(ebs_total * ((disk_avg) / 100)) + '%', Backup, iam_role, private_key, Id, Link]

with ThreadPoolExecutor() as executor:
    rows = list(executor.map(mainFunction, instances))

with open(filename, 'w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(column_names)
    writer.writerows(rows)

print(f"CSV file '{filename}' created successfully.")

email_subject = "AWS"
email_body = "FALCO AWS / Data"
email_recipient = "marcos.pozzetti@veolia.com matteo.onorato@veolia.com umberto.zuccala@veolia.com davide.benedetto@veolia.com andrea.fiscella@veolia.com"
email_command = f"echo '{email_body}' | mutt -s '{email_subject}' -a {filename} -- {email_recipient}"

subprocess.run(email_command, shell=True)

print("CSV sent")
```
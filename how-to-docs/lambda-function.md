## Simple Lambda Function to Schedule Instance Cleanup

Ensure you are accessing the correct AWS account, this example is for the
DSP playground.

---

### Create a role

- create a role with ability to describe and stop instances
- name it _lambda_cleanup_
- select AWS lambda for role type
and do not attach a policy
- create role


reselect the role and add the inline policy below 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "ec2:Stop*"
            ],
            "Resource": "*"
        }
    ]
}
```
---

### Create a Lambda function

- select lambda and from blueprint select the blank function
- name the function and select python2.7 

Paste the following python snippet into the box


```
import boto3
import logging

# https://www.nimbo.com/blog/automating-ec2-tasks-with-aws-lambda/
# setup simple logging for INFO
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# define the connection
ec2 = boto3.resource('ec2')

def lambda_handler(event, context):
    filters = [{
            'Name': 'tag:Env',
            'Values': ['playground*']
        },
        {
            'Name': 'instance-state-name', 
            'Values': ['running']
        }
    ]
    
    # filter the required instances
    instances = ec2.instances.filter(Filters=filters)

    RunningInstances = [instance.id for instance in instances]
    print RunningInstances 
    
    # make sure there are actually instances to shut down.
    if len(RunningInstances) > 0:
        #perform the shutdown
        shuttingDown = ec2.instances.filter(InstanceIds=RunningInstances).stop()
        print shuttingDown
    else:
        print "Nothing to stop"
        
```

- select the existing role created above _lambda_cleanup_
- increase the timeout to 15 seconds
- create the function using other default values

---

### Test and Schedule as a Job

The function may be tested if safe to do so. An alternative is to temporarily change
the filter to match only terminateable instances. eg ` 'Values': ['playground-arpat*'] `

If you wish to schedule this function as similar to a cron job, click the trigger
tab.

- select add trigger
- click the dotted box and select the trigger type select CloudWatch Events -schedule
- fill the boxes and select Schedule expression of cron

eg. this would set the trigger for every Friday at 6pm.
` cron(0 18 ? * FRI *) `

 




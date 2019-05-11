---
layout: post
title:  "Integrate ArcherySec + OWASP ZAP in Jenkins CI/CD Pipeline"
author: anand
categories: [ Tutorial ]
image: assets/images/jenkins/archery+jenkins.png
featured: true
hidden: false
comments: true
---

Continuous Integration / Continuous Deployment (CI/CD) processes allow software developers to detect problems early in the development lifecycle and improve productivity with automation. 

<em> "[Jenkins](http://jenkins.io) is a popular open-source continuous integration solution that helps teams manage the automation of software build, as well as monitor the execution of external jobs that supports the software build."</em>

When you run security testing in your CI/CD pipeline, you want to store vulnerability data in centralize way and manage them easily for your every single pipeline. DevOps teams are facing challenges to have a central place where they can visualize vulnerabilities transparently. Vulnerability Management is one of the challenging part for every organization.

In this article we'll be going learn how to integrate Archery tool in your jenkins CI/CD pipeline.

<blockquote ><em>"ArcherySec is an open source vulnerability assessment and management tool that helps developer and pentester to perform vulnerability assessment and manage vulnerabilities."</em></blockquote>

Archery has [API](https://developers.archerysec.com/) that interact with Archery tool and automate vulnerability assessment process. 

[Archerysec-cli](https://github.com/archerysec/archerysec-cli) uses the API to interact Archery tool from console. We often use archerysec-cli in our CI/CD pipeline steps to perform scan and feed vulnerability data into Archery tool. 

### Configure Lab Environment:

Requirements:
- Docker & docker-compose

Steps to configure Lab Environment.
```
$ git clone https://github.com/archerysec/jenkins_blog.git
$ cd jenkins_blog
$ run docker-compose up --build
```

We have written small scripts and Infrastructure as a code that allow you to spin up environment on your system. 


docker-compose.yml

```bash
version: '3.6'

services:
  jenkins:
    build: ./jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home
    expose:
      - "8080"
      - "50000"
    container_name: jenkins
    
  db:
    image: postgres:10.1-alpine
    volumes:
      - ./dbdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=archerysec
      - POSTGRES_PASSWORD=archerysec
      - POSTGRES_USER=archerysec

  archerysec:
    image: archerysec/archerysec
    ports:
      - "8000:8000"
    expose:
      - "8000"
    depends_on:
      - db
    links:
      - db:db
    environment:
      - DB_PASSWORD=archerysec
      - DB_USER=archerysec
      - DB_NAME=archerysec
      - DB_HOST=db
      - DJANGO_SETTINGS_MODULE=archerysecurity.settings.development
      - DJANGO_SECRET_KEY=${DJANGO_SECRET_KEY:-"SETME"}
      - DJANGO_DEBUG=1
    container_name: archerysec
   
  zaproxy:
    image: owasp/zap2docker-stable
    command: zap.sh -daemon -host 0.0.0.0 -port 8090 -config api.disablekey=true -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true
    ports:
      - "8090:8090"
    expose:
    # ZAP is running on 8090, we want it to be accessible by our tools
      - "8090"
    links:
      - archerysec
    container_name: zapscanner
```

Once containers are up and running you could check whether they are accessible or not by accessing the below URL’s.

ArcherySec: http://your_system_ip_address:8000<br>
Jenkins: http://your_system_ip_address:8080<br>
OWASP ZAP: http://your_system_ip_address:8090

#### Connect ZAP with ArcherySec

- Open ArcherySec portal.
- Go to settings page http://your_system_ip_address:8000/webscanners/setting/.
- Edit ZAP Settings.
- Provide ZAP API Host & ZAP API Port


<center><div class="img-border" style="width: 70%;"><img src="/assets/images/jenkins/zap_setting_archery.png"></div></center>

<br>


Next thing we need to install archerysec-cli tool on jenkins server.

```bash
$ pip install archerysec-cli
Or 
$ git clone https://github.com/archerysec/archerysec-cli.git
$ cd archerysec-cli
$ pip install -r requirements.txt

# Install jq tool
sudo apt-get install jq

```

We can create project using CLI commands:

```bash
$ archerysec-cli -s http://127.0.0.1:8000 -u admin -p admin --createproject \
 --project_name=test_project --project_disc="test project" --project_start=2018-01-11 \
  --project_end=2018-01-11 --project_owner=anand
```
Output:
```bash
{"message": "Project Created", "project_id": "8b4303bc-2cd6-4212-8801-231e8b10be4d"}
```

We can launch scan(s) using CLI:

```bash
$ archerysec-cli -s http://127.0.0.1:8000 -u admin -p admin \
 --zapscan --target_url=http://demo.testfire.net \
  --project_id=8b4303bc-2cd6-4212-8801-231e8b10be4d
```
Output:
```bash
{"message": "Scan Launched", "scanid": "23a9ea3f-088d-4155-9585-5eb1ae483ef8"}
```

Get the scan status: 

```bash
$ archerysec-cli -s http://127.0.0.1:8000 -u admin -p admin \
 --zapscanstatus --scan_id=23a9ea3f-088d-4155-9585-5eb1ae483ef8
```
Output:
```bash
[{"scan_url": "http://demo.testfire.net", "medium_vul": null, "rescan_id": null, "
date_time": "2019-02-27T04:43:40.254359Z", "vul_status": 1, "low_vul": null,
 "rescan": "No", "vul_num": "", "scan_scanid": "23a9ea3f-088d-4155-9585-5eb1ae483ef8",
  "total_dup": null, "project_id": "8b4303bc-2cd6-4212-8801-231e8b10be4d", 
  "high_vul": null, "total_vul": null}]
```
Get the scan results:

```bash
$ archerysec-cli -s http://127.0.0.1:8000 -u admin -p admin --zapscanresult \
  --scan_id=049b8207-f092-4f6f-b375-92ed541dc662
```

Let’s automate these steps using bash and python scripts.
- Install archerysec-cli
- Create Project and get the Project ID
- Launch ZAP Scan and retrieve scan ID
- Check the scan status.
- Grab the vulnerability statistics.

archerysec_script.py


```python
from pyArchery import api
from optparse import OptionParser, OptionGroup
import json
import time

parser = OptionParser()

group = OptionGroup(parser, "",
                   "")
parser.add_option_group(group)
group = OptionGroup(parser, "Archery Scan status",
                   "Upload multiple scanners reports"
                   )
group.add_option("--scanner",
                help="Input input scanner name i.e zap_scan, arachni_scan",
                action="store")
group.add_option("--scan_id",
                help="Input Scan Id",
                action="store")
group.add_option("--username",
                help="Input ArcherySec Username",
                action="store")
group.add_option("--password",
                help="Input ArcherySec Password",
                action="store")
group.add_option("--host",
                help="Input ArcherySec Host",
                action="store")
group.add_option("-r", "--high",
                help="Numbers of issue",
                action="store")
group.add_option("-m", "--medium",
                help="Numbers of issue",
                action="store")
(args, _) = parser.parse_args()

def archery_host():
   # Setup archery connection
   archery = api.ArcheryAPI(args.host)
   return archery

def archery_auth():
   # Set Archery url
   archery = archery_host()

   # Provide Archery Credentials for authentication.
   authenticate = archery.archery_auth(args.username, args.password)

   # Collect Token after authentication
   token = authenticate.data
   for key, value in token.viewitems():
       token = value
   return token

# Get the scan result
if args.scanner == 'zap_scan':
   time.sleep(5)
   archery = archery_host()
   web_scan_result = archery.zap_scan_status(
       auth=archery_auth(),
       scan_id=args.scan_id,
   )
   results = web_scan_result.data_json()
   j_result = json.loads(results)
   for j in j_result:
       scan_status = j['vul_status']
       print scan_status
       while (int(scan_status) < 100):
           web_scan_result = archery.zap_scan_status(
               auth=archery_auth(),
               scan_id=args.scan_id,
           )
           results = web_scan_result.data_json()
           j_result = json.loads(results)
           try:
               for j in j_result:
                   scan_status = j['vul_status']
           except Exception as e:
               scan_status = 100
           time.sleep(10)
           print "Scan Status", scan_status
       time.sleep(60)
      
       web_scan_result = archery.zap_scan_status(
           auth=archery_auth(),
           scan_id=args.scan_id,
       )
       results = web_scan_result.data_json()
       j_result = json.loads(results)
       for j in j_result:
           total_vul = j['total_vul']
           high_vul = j['high_vul']
           medium_vul = j['medium_vul']
           low_vul = j['low_vul']
       print "Total Vul", total_vul
       print "Total High", high_vul
       print "Total Medium", medium_vul
       print "Total Low", low_vul
       if int(high_vul) >= int(args.high):
           fail = "FAILURE"
           print "Coz total high Vulnerability", high_vul
       elif int(medium_vul) >= int(args.medium):
           fail = "FAILURE"
           print "Coz total Medium Vulnerability", medium_vul
       else:
           fail = "SUCCESS"
           print "Test Passed"

       print fail

```

zapscan.sh

```bash
#
pip install archerysec-cli

DATE=`date +%Y-%m-%d`
TARGET_URL=http://demo.testfire.net
ARCHERY_USER=admin
ARCHERY_PASS=admin

export PROJECT_ID=`archerysec-cli -s ${ARCHERY_HOST} -u ${ARCHERY_USER} -p ${ARCHERY_PASS} --createproject --project_name=DevSecOps --project_disc=PROJECT_DISC --project_start=${DATE}  --project_end=${DATE} --project_owner=test_project | tail -n1 | jq '.project_id' | sed -e 's/^"//' -e 's/"$//'`
export SCAN_ID=`archerysec-cli -s ${ARCHERY_HOST} -u ${ARCHERY_USER} -p ${ARCHERY_PASS} --zapscan --target_url=''${TARGET_URL}'' --project_id=''$PROJECT_ID'' | tail -n1 | jq '.scanid' | sed -e 's/^"//' -e 's/"$//'`
echo "scan id......" $SCAN_ID
python /var/jenkins_home/archery_script.py --scanner=zap_scan --scan_id=$SCAN_ID --username=${ARCHERY_USER} --password=${ARCHERY_PASS} --host=${ARCHERY_HOST} --high=10 --medium=15
export job_status=`python /var/jenkins_home/archery_script.py --scanner=zap_scan --scan_id=$SCAN_ID --username=${ARCHERY_USER} --password=${ARCHERY_PASS} --host=${ARCHERY_HOST} --high=10 --medium=15`]
if [ -n "$job_status" ]
then
   echo "Build Sucess"
else
 echo "BUILD FAILURE: Other build is unsuccessful or status could not be obtained."
 exit 100
fi
```

Now run all these script on Jenkins CI pipeline.

- Login into Jenkins server.
- Create new job.
- Give the name of the job and select as pipeline.
- Go to pipeline section and copy paste  below code.

```groovy

pipeline {
  agent any
  stages {
        stage('DAST') {
          parallel {
            stage('OWASP ZAP') {
              agent any
              steps {
                sh '''
                 export ARCHERY_HOST=http://your_system_ip_address:8000
                     bash /var/jenkins_home/zapscan.sh
                  '''
              }
            }
          }
        }
    }
}
```

- Replace ARCHERY_HOST value with your system IP address.

<center><div class="img-border" style="width: 70%;"><img src="/assets/images/jenkins/pipeline_content.png"></div></center>

<br>

- Apply and save.
- Now click on Build Now.
- Go to Console Output and notice that the scripts are running.

<center><div class="img-border" style="width: 70%;"><img src="/assets/images/jenkins/jenkins_ruuning_zap.png"></div></center>
<br>

Now open ArcherySec and go to Dynamic Scans > ZAP Scans. Notice that the scan are running and given the running status % of scan.


<center><div class="img-border" style="width: 70%;"><img src="/assets/images/jenkins/blog_archery_running.png"></div></center>

<br>

Jenkins will now run OWASP ZAP using ArcherySec at your desired frequency and will tell you whether the build failed or succeeded. In a bigger setup, ArcherySec will be part of your build process. You can set up notifications and customize Jenkins as per your needs.
You can use a wide variety of other configurations to make your collection more dynamic. 

#### Conclusion

Following the steps above, you will be able to set up a continuous integration process that includes ArcherySec automated Dynamic OWASP ZAP tests for the application under scan. Each commit will trigger an automated test run. Once the test run has finished, you’ll able to manage vulnerabilities using ArcherySec Tool.

#### Learn More

- [http://www.archerysec.com/](http://www.archerysec.com/)
- [https://github.com/archerysec/archerysec](https://github.com/archerysec/archerysec)
- [https://docs.archerysec.com/](https://docs.archerysec.com/)
- [https://developers.archerysec.com/](https://developers.archerysec.com/)




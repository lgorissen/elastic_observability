# Lab 0: Elastic Observability trial instance

Goal of Lab 0 is to acquire an Elastic Observability trail instance. It takes only a couple of steps.

Along the way, there is some information that has to be collected for the later labs.

## Create an account

Go to https://www.elastic.co/ :
![Elastic home](img/elastic-observability_000.png)

click on the *Login* button in the right upperhand corner to go to Login:
![Login screen](img/elastic-observability_001.png)

Click *Sign up*: 
![Sign up screen](img/elastic-observability_002.png)

Enter your email and password and click *Start free trial*. That will bring you to this screen: 
![Sign up screen](img/elastic-observability_003.png)

Now, verify your account from the e-mail that you received: click Verify and Accept.
![Verificiation em-mail](img/elastic-observability_004.png)

Now, on the already opened window go to the Cloud root:
![Verificiation em-mail](img/elastic-observability_005.png)

Click *Start your free trial* 
![Elastic Observability](img/elastic-observability_006.png)

And select Observability:
![Elastic Observability - settings](img/elastic-observability_007.png)

Look at the 'Deployment settings', the 'deployment name' and then click 'Create deployment' and wait for your `elastic`  credentials:

![Elastic credentials](img/elastic-observability_008.png)

Save the credentials, you will need them later on ...

| item       | value |
|------------|-------|
|user        | elastic |
|password    | ...|

Wait until your trial instance has been created. This looks like:

![Trial instance created](img/elastic-observability_009.png)

Now, scroll down the page:

![trial deployment](img/elastic-observability_010.png)

... and collect the following information:

| item       | value |
|------------|-------|
|Cloud ID    | elastic-observability-deployment:ZWFz ... |


Finally, go to the APM section:

![APM section](img/elastic-observability_011.png)

... and collect the following information

| item       | value |
|------------|-------|
| endpoint | https://a341003.....29db18565a.apm.eastus2.azure.elastic-cloud.com | 
| APM Server secret token | 2X..............F5 | 

Now, your Elastic Observability set-up is ready for the remaining labs.

# Maltego Lambda How To
When building Maltego transforms one important piece of the equation is the transformation server.

One can get started building a transform server quickly by following the instructions here:
[Transform Host Server Setup](https://docs.paterva.com/en/developer-portal/tds-transforms/transform-host-server-setup/)

The following How To is designed to be an alternative to the method above so that one can eliminate the actual server by using Amazon’s Lambada, a service that runs code without the server.

The guide leverages heavily on the Zappa project.  
Zappa is designed to build and deploy Python WSGI scripts to AWS Lambda.
[GitHub - Miserlou/Zappa: Serverless Python Web Services](https://github.com/Miserlou/Zappa)

*WARNING*, I don’t claim to be an expert in any of these areas so if you see a piece that is wrong or could be improved please let me know.

Step 1:  You’ll need to sign up for an amazon account and install Boto3.
[Quickstart — Boto 3 Docs 1.4.7 documentation](http://boto3.readthedocs.io/en/latest/guide/quickstart.html#installation)
``` py
pip install boto3
aws configure
```

When configuring your amazon AWS a credential file will be generated @ ~/.aws/config

``` text
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

Step 2:  You’ll need to install Zappa.  There is excellent documentation on their GitHub site
[GitHub - Miserlou/Zappa: Serverless Python Web Services](https://github.com/Miserlou/Zappa#initial-deployments)
``` text 
pip install zappa
zappa init
```

During the init a series of questions will be asked.  Here is how I respond to those questions but your responses may vary.

The first step names the end-point API.  I create my first end-point something_dev in this case maltego_dev
![](/images/8045552B-5ECD-4EA0-85C1-86F4A0851CBF.png)

The second step reads the AWS credential file that was created when installing AWS boto3
![](/images/BAA28B24-C536-4000-8E65-469A8553092B.png)

The second step prompts you to name your s3 bucket where you source code will be uploaded to AWS
![](/images/A800D5A0-7F63-45DF-9443-6CFBE4441852.png)

Naming your app
![](/images/409B834B-B88F-4507-979F-FA6AF39502F5.png)

Region availability
![](/imagesA4B8E45F-76A9-4D81-BDBF-492BB986C874.png)

And your output will a Zappa setting file named zappa_settings.json

``` json
{
    "maltego_dev": {
        "app_function": "maltego_transforms.app",
        "aws_region": "us-east-2",
        "profile_name": "somethinghere",
        "s3_bucket": "maltego-serverless-transforms"
    }
}
```

Once the settings file is complete you can deploy the app to AWS and some magic happens
``` bash
zappa deploy maltego_dev
```

![](/images/241E8BF5-77F6-458C-8E35-B26848A4F739.png)

At this point you have a empty Flask application sitting on Lambda.  Go ahead and test out the end point as this will be what you use when setting up your transform.  You should see a debug message but that will be taken care of when we start building our transforms.

Step 3:  Building Flask Based Transforms
The following is flask based with two functions.  

The root / directory will return a simple line of text that can be used to validate your Flask app is working and serving content.

The next /stuff2stuff is the transform
This endpoint will be used in the Paterva TDS
https://a3mwnt1kz9.execute-api.us-east-2.amazonaws.com/maltego_dev/stuff2stuff

``` python
import requests
import json

from flask import Flask
from flask import request

from Maltego import *

app = Flask(__name__)

@app.route('/')
def hello():
	return "Maltego Transformations Ready Without a Server To Crash"

@app.route('/stuff2stuff', methods=['GET', 'POST'])
def stuff():
	#This line takes the entity that is passed on creates a Maltego MSG object so you can access the values
	maltego = MaltegoMsg(request.get_data()) 
	TRX = MaltegoTransform()
	TRX.addEntity("maltego.Phrase", maltego.Value)
	return TRX.returnOutput()

if __name__ == '__main__':
	app.run()
```

Step 4:  Configure TDS to point to end-point

Adding the following to the Paterva TDS Server.  You may need to refresh your Maltego Client to pull in the new transforms

—Click Do not test URL—
https://a3mwnt1kz9.execute-api.us-east-2.amazonaws.com/maltego_dev/stuff2stuff
![](/images/Maltego%20Lambda%20How%20To/67177052-903E-40D4-925B-C56D75021969.png)

Confirm Transforms Is Working
![](/images/Maltego%20Lambda%20How%20To/86DE640F-12D5-4110-ACA3-D3AC912C3FB3.png)




#Lambda
#AWS
#Maltego

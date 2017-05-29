# Slack Signup with AWS Lambda

Serverless slack sign-up with AWS Lambda, in Python.
Inspired by [Serverless Slack Invite Service](https://github.com/serverless-london/serverless-slack-invite).

This creates a serverless service for users to sign up for invitations to your Slack. Similar to [slackin](https://github.com/rauchg/slackin), but serverless, that is, no need to run (and pay for) a [virtual] server. The invitation function will run on demand and you only pay per use. 

As a learning exercise for AWS Lambda and serverless, this 
project is a **take one**: it uses AWS "raw", with AWS all configurations done with `aws` CLI and some console, to understanding the fundamentals of Lambda and API Gateway, and to appreciate the need for a good framework, like [serverless.com](https://serverless.com).

Disclaimer: The README.md are not (yet) full blown instructions but more "the notes to myself", *PG rated*, read on your risk.

## Lambda

* API for Slack invite, from [the document of undocumented API](https://github.com/ErikKalkoken/slackApiDoc/blob/master/users.admin.invite.mdCode)

* How to obtain Slack Auth token (the hard and easy way): 
https://github.com/StackStorm-Exchange/stackstorm-slack#obtaining-auth-token

* To run Lambda "lean" and make local testing match remote execution, I don't use `virtualenv` and manage the path from the code, see [hint here](http://forum.serverless.com/t/aws-python-function-dependencies-load/451/4)

    ```
    pip install -t ./site-packages/ -r requirements.txt
    ```
    Note that `setup.cfg` is required per [this AWS hint](http://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html).
    
#### AWS Lambda cheat sheet: 

* Check what we get so far: 
    ```
    aws --region=us-east-1 lambda list-functions
    ```


* Zip up the file:

    ```
    zip -X -r ../deploy/invite.zip * 
    ```

* Create a lambda function
    
    ```
    aws lambda create-function \
    --region us-east-1 \
    --function-name SlackInviteAWS \
    --zip-file fileb://../deploy/invite.zip \ 
    --handler invite.lambda_handler \ 
    --runtime python2.7 \
    --role arn:aws:iam::053075847820:role/lambda_basic_execution
    ```

    Note that the role need to exist there and have a policy to create and access CloudWatch log streams. You may find it easier to use AWS console to create one. If your function works but you can't see the logs, blame it on the role.

* Sure enough I've messed up something so fix the code, re-zip the zip, and update a function:

    ```
    aws lambda update-function-code \
    --region us-east-1 \
    --function-name SlackInviteAWS \
    --zip-file fileb://../deploy/invite.zip
    ```

* To update the function settings, (e.g., environment variables)

    ```
    aws lambda update-function-configuration \
    --region us-east-1 \
    --function-name SlackInviteAWS \
    --environment '{"Variables":{"SLACK_TEAM":"dz-test", "SLACK_TOKEN":"xxxx-111111100111-111111111111-111000111000-111eeeaa11"}}'
    ```

## API Gateway
Follow the step by step instructions in the [Step-by-step guide](http://docs.aws.amazon.com/lambda/latest/dg/with-on-demand-https-example-configure-event-source.html). At some point say **Fuck it I go [serverless.com](https://serverless.com/framework/docs/providers/aws/guide/quick-start/)**

The notes on this section are ways too long to post here and generally repeat the Step-by-step guide. There are few traps I got into, so I may still post it later. 

* CURL to test: 

    ```
    $ curl -X POST -d '{ "email": "your.email+Trump@gmail.com", "first_name": "Donald" }' --url https://c27hhqorea.execute-api.us-east-1.amazonaws.com/prod/slack_invite/
    [true, "Invite sent"]
    ```

* The Lambda function logs are in CloudWatch logs (update URL with your Lambda function name:  
`https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logStream:group=/aws/lambda/SlackInviteAWS;streamFilter=typeLogStreamPrefix`

## Deploy web UI
* First, you'll need to enable CORS on the API, see [How to CORS](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)

    > Don't forget to deploy the newly created `OPTIONS` method!

    Test that it works:
 
    ```
    url -X OPTIONS --url https://c27hhqorea.execute-api.us-east-1.amazonaws.com/prod/slack_invite/
    ```







 

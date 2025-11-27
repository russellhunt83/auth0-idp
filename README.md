# Disclaimer
I take no responsibility for the security of your AWS account, or any misconfigurations in your Auth0 tenant. This code is provided as is (see overview for intended purpose). You **must** carry out your own fully comprehensive security review before using custom built applications, scripts or code to manage the security of your own cloud platforms. 

This script does not deliberately log, expose, or transfer security information outside of your network other than to the systems referred to in this Readme and their subsidiary. Please ensure you have fully reviewed the platforms before using this code or following this README. You can find out more at https://auth0.com/ 

I am not affiliated, work for or are promoting the use of Auth0 as a service. This code is provided 'as-is', and is shared as something that I use to keep my personal coding hobby environments secured.

# Overview
Short guide to configure Auth0 as an identity provider (IdP). This script allows you to sign-in to ontain temporary AWS credentials using Auth0's free SSO Applications. Reducing the need to create IAM Credentials. 

- Enables single sign-on (SSO) to the AWS CLI V2 via Auth0. 
- Readme covers creating an Auth0 SSO

## Prerequisites
- Free Auth0 tenant and dashboard access - https://auth0.com
- AWS account with IAM permissions to create Identity providers and IAM roles - https://console.aws.amazon.com (SEE DISCLAIMER)


# Steps

1. Create an application in Auth0
    - In Auth0 Dashboard → Applications → Applications → Create Application.
    - Choose "Native" 
    - In "settings" got to Application URIs and add "http://127.0.0.1:53682/callback" (NOTE: If you are setting a non-default port when calling gimme-auth0-creds, remember to change the port in the URL example from 53682 to the port you provided in the --port parameter)
    - Take a note of the application ID for step 2
    

2. Create a OpenID Connect provider in AWS IAM
    - AWS Console → IAM → Identity providers → Create provider.
    - Provider type: OpenID Connect
    - Provider URL: {your full qualified Auth0 domain name WITH the trailing slash}
    - Audience: {The application ID that you created in step 1}
    - Create the provider 


3. Create IAM role(s) for SSO
    Once the OpenConnect Provider is setup, you can assign a role to it:
    - In the provider entry click "Assign Role"
    - Choose an existing role or create a new one
    - Apply RBAC (Role Based Access Control). Optionally: for this example try providing Read only to an existing S3 bucket



4. Test SSO
    Open the .auth0_aws_login_config.default
    ```
    [default]
        tenant_domain = yourdomain.auth0.com
        auth0_client_id = {YOUR_CLIENT_ID}
        aws_role_arn = arn:aws:iam::{AWS_ACCOUNT_ID}:role/{ROLE}
        aws_profile = default
        aws_region = {aws_region}
    ```

    1. tenant_domain = the tenant URL provided by Auth0
    2. auth0_client_id = The client ID provided in the app you created in step 1
    3. aws_role_arn = the role you created in step 2
    4. aws_profile = Unless you have multiple apps, it's fine to leave as default 
    5. aws_region = the AWS naming convention format, e.g. eu-west-1
    
    Save this file to your home directory as .auth0_aws_login_config or pass the --config parameter with the full path to the config file.

    ```
    chmod +x ./gimme-auth0-creds
    ```

    Finally, from the directory that has the gimme-auth0-creds file :
    ```
    python3 -m venv .venv \
    source ./.venv/bin/activate \
    pip install -r requirements.txt \
    ./gimme-auth0-creds
    ```

    Follow the browser prompts.

    If the login succeeded, you should have an AWS Role in your ~/.aws/credentials file with temporary credentials that can inherit the role on AWS. 

    For example if your gave the role read access to S3, you can test it by running
    
    ```
    aws s3 ls
    ```

# Extra Information
OpenID Connect will log Role activity in AWS. If you want to see who used the role, you can use Auth0 logging. Applications are a free part of Auth0 and useful for testing deployments in AWS without having to create IAM Credentials. 

Every user in your Auth0 tenant who knows the Application ID for this app will be able to log onto it. You can use flows in Auth0 to control this, but that is beyond the scope of this application.

I do not recommend this application for use in larger organisations. SAML Authentication from Auth0 provides much more granular access.

Credit to Nike - gimme-aws-creds for Okta (https://github.com/Nike-Inc/gimme-aws-creds) for the foundational idea for this script. See https://auth0.com for more information.
# User Authentication: Identity-Aware Proxy

## content
- How to write and deploy a simple App Engine app using Python
- How to enable and disable IAP to restrict access to your app
- How to get user identity information from IAP into your app
- How to cryptographically verify information from IAP to protect against spoofing

## What is Identity-Aware Proxy?

Identity-Aware Proxy (IAP) is a Google Cloud service that intercepts web requests sent to your application, authenticates the user making the request using the Google Identity Service, and only lets the requests through if they come from a user you authorize.
In addition, it can modify the request headers to include information about the authenticated user.

### Download the code
Click the command line area in the Cloud Shell so you can type commands.

Download the code from a public storage bucket and then change to the code folder:
```yaml
gsutil cp gs://spls/gsp499/user-authentication-with-iap.zip .
```
```yaml
unzip user-authentication-with-iap.zip
```
```yaml
cd user-authentication-with-iap
```
This folder contains one subfolder for each step of this lab. You will change to the correct folder to perform each step.

## 1. Deploy the application and protect it with IAP
This is an App Engine Standard application written in Python that simply displays a "Hello, World" welcome page. We will deploy and test it, then restrict access to it using IAP.

Review the application code
Change from the main project folder to the 1-HelloWorld subfolder that contains code for this step.
```yaml
cd 1-HelloWorld
```
The application code is in the main.py file. It uses the Flask web framework to respond to web requests with the contents of a template. That template file is in templates/index.html, and for this step contains only plain HTML. A second template file contains a skeletal example privacy policy in templates/privacy.html.

There are two other files: requirements.txt lists all the non-default Python libraries the application uses, and app.yaml tells Google Cloud that this is a Python App Engine application.

You can list each file in the shell using the cat command, as in:
```yaml
cat main.py
```
Or you can launch the Cloud Shell code editor by clicking the Pencil icon at the top right-hand side of the Cloud Shell window, and examine the code that way.

You do not need to change any files for this step.

**Deploy to App Engine**
1. Update python runtime to python313.
```yaml
sed -i 's/python37/python313/g' app.yaml
```
2. Deploy the app to the App Engine Standard environment for Python.
```yaml
gcloud app deploy
```
![2](https://github.com/user-attachments/assets/915abae8-ff81-4abd-a2cb-41b9adbedcdf)

![3](https://github.com/user-attachments/assets/1464dc7f-0d49-4526-b0f4-535bf16ad27e)

3. Select a region REGION.
4. When you are asked if you want to continue, enter Y for yes.
Note: If you get a Gaia propagation related error message, re-run the gcloud app deploy command.
In a few minutes the deployment completes. You will see a message that you can view your application with gcloud app browse.
5. Enter that command:
```yaml
gcloud app browse
```
6. Click the displayed link to open it in a new tab, or copy it to a manually opened new tab if necessary. Since this is the first time this app is run, it will take a few seconds to appear while a cloud instance is started, and you should see the following window.

![4](https://github.com/user-attachments/assets/bd8b0d4d-eeef-4150-98b1-c31a42082977)

### Restrict access with IAP
1. In the cloud console window, click the Navigation menu Navigation menu icon > Security > Identity-Aware Proxy.
2. Click Enable API.
3. Click Go to Identity Aware Proxy.
4. To configure your project's OAuth consent screen, go to the OAuth consent screen
5. Click Get Started.
6. For App name, enter IAP Example.
7. Click User support email, and then click the student email and then click Next.
8. For Audience, select Internal, and then click Next.
Users with access to the project should be able to log in to the app.
9. On the left panel of the lab instructions, copy the Username.
10. For Contact information, paste the copied username.
11. Click Next.
12. Click Checkbox to agree the User Data Policy and click Continue and then click Create.
The consent screen is now set up.
13. In Cloud Shell, run this command to disable the Flex API:
```yaml
gcloud services disable appengineflex.googleapis.com
```
Note: App Engine has its standard and flexible environments which are optimized for different application architectures. Currently, when enabling IAP for App Engine, if the Flex API is enabled, Google Cloud will look for a Flex Service Account. Your lab project comes with a multitude of APIs already enabled for the purpose of convenience. However, this creates a unique situation where the Flex API is enabled without a Service Account created.
Return to the Identity-Aware Proxy page and refresh it. You should now see a list of resources you can protect.
Click the toggle button in the IAP column in the App Engine app row to turn IAP on.

The domain will be protected by IAP. Click Turn On.

## Test that IAP is turned on
1. Open a browser tab and navigate to the URL for your app. A Sign in with Google screen opens and requires you to log in to access the app.
2. Sign in with the account you used to log into the console. You will see a screen denying you access.
You have successfully protected your app with IAP, but you have not yet told IAP which accounts to allow through.
3. Return to the Identity-Aware Proxy page of the console, select the checkbox next to App Engine app, and see the App Engine sidebar to the right.
Each email address (or Google Group address, or Workspace domain name) that should be allowed access needs to be added as a Member.
4. Click Add Principal.
5. Enter your Student email address.
6. Then, pick the Cloud IAP > IAP-Secured Web App User role to assign to that address.
You may enter more addresses or Workspace domains in the same way.
7. Click Save.

![5](https://github.com/user-attachments/assets/deaa6031-738a-43a7-9e60-489ee32bec68)

### Test access
Navigate back to your app and reload the page. You should now see your web app, since you already logged in with a user you authorized.

If you still see the "You don't have access" page, IAP did not recheck your authorization. In that case, do the following steps:

Open your web browser to the home page address with /_gcp_iap/clear_login_cookie added to the end of the URL, as in https://iap-example-999999.appspot.com/_gcp_iap/clear_login_cookie.
You will see a new Sign in with Google screen, with your account already showing. Do not click the account. Instead, click Use another account, and re-enter your credentials.
Note: It takes a minute for the role change to take effect. If the page still shows the "You don't have access" message after following the previous steps, wait a minute and try refreshing the page.
These steps cause IAP to recheck your access and you should now see your application's home screen.

If you have access to another browser or can use Incognito Mode in your browser, and have another valid Gmail or Workspace account, you can use that browser to navigate to your app page and log in with the other account. Since that account has not been authorized, it will see the "You Don't Have Access" screen instead of your app.

## 2. Access user identity information
Once an app is protected with IAP, it can use the identity information that IAP provides in the web request headers it passes through. In this step, the application will get the logged-in user's email address and a persistent unique user ID assigned by the Google Identity Service to that user. That data will be displayed to the user in the welcome page.

In Cloud Shell, change to the folder for this step:
```yaml
cd ~/user-authentication-with-iap/2-HelloUser
```

### Deploy to App Engine
1. Update python runtime to python313.
```yaml
sed -i 's/python37/python313/g' app.yaml
```
2. Since deployment takes a few minutes, start by deploying the app to the App Engine Standard environment for Python:
```yaml
gcloud app deploy
```
When you are asked if you want to continue, enter Y for yes.

In a few minutes the deployment should complete. While you are waiting you can examine the application files as described below

![2](https://github.com/user-attachments/assets/4ebea7dd-4b7f-4941-86d3-e0cf0315dea2)

### Examine the application files
This folder contains the same set of files as seen in the previous app you deployed, 1-HelloWorld, but two of the files have been changed: main.py and templates/index.html. The program has been changed to retrieve the user information that IAP provides in request headers, and the template now displays that data.

There are two lines in main.py that get the IAP-provided identity data:
**user_email = request.headers.get('X-Goog-Authenticated-User-Email')
user_id = request.headers.get('X-Goog-Authenticated-User-ID')**

The X-Goog-Authenticated-User- headers are provided by IAP, and the names are case-insensitive, so they could be given in all lower or all upper case if preferred. The render_template statement now includes those values so they can be displayed:
**page = render_template('index.html', email=user_email, id=user_id)**

The index.html template can display those values by enclosing the names in double curly braces:

Hello, {{ email }}! Your persistent ID is {{ id }}.
As you can see, the provided data is prefixed with accounts.google.com, showing where the information came from. Your application can remove everything up to and including the colon to get the raw values if desired.

### Test the updated IAP
Going back to the deployment, when it is ready, you will see a message that you can view your application with gcloud app browse.

1. Enter that command:
```yml
gcloud app browse
```
![3](https://github.com/user-attachments/assets/c40fedb7-3dd6-4842-8b8a-1db0c538d917)

2. If a new tab does not open on your browser, copy the displayed link and open it in a new tab normally. 

You may need to wait a few minutes for the new version of your application to replace the prior version. Refresh the page if needed to see a page similar to the above.

![1](https://github.com/user-attachments/assets/4d6b3a48-3be7-4349-8176-564706ce510a)

## Turn off IAP
What happens to this app if IAP is disabled, or somehow bypassed (such as by other applications running in your same cloud project)? Turn off IAP to see.
1. In the cloud console window, click Navigation menu > Security > Identity-Aware Proxy.
2. Click the IAP toggle switch next to App Engine app to turn IAP off. Click TURN OFF.
You will be warned that this will allow all users to access the app.
3. Refresh the application web page. You should see the same page, but without any user information:

![4](https://github.com/user-attachments/assets/0d920b55-c607-4cd3-b98c-9348f5ec002d)

Since the application is now unprotected, a user could send a web request that appeared to have passed through IAP. For example, you can run the following curl command from the Cloud Shell to do that (replace <your-url-here> with the correct URL for your app):
```yml
curl -X GET <your-url-here> -H "X-Goog-Authenticated-User-Email: totally fake email"
```
The web page will be displayed on the command line, and look like the following:

<!doctype html>
<html>
<head>
  <title>IAP Hello User</title>
</head>
<body>
  <h1>Hello World</h1>
  <p>
    Hello, totally fake email! Your persistent ID is None.
  </p>
  <p>
    This is step 2 of the <em>User Authentication with IAP&lt;/em&gt;
    codelab.
 &lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;
</em>
There is no way for the application to know that IAP has been disabled or bypassed. For cases where that is a potential risk, Cryptographic Verification shows a solution.

## 3. Use Cryptographic Verification
If there is a risk of IAP being turned off or bypassed, your app can check to make sure the identity information it receives is valid. This uses a third web request header added by IAP, called X-Goog-IAP-JWT-Assertion. The value of the header is a cryptographically signed object that also contains the user identity data. Your application can verify the digital signature and use the data provided in this object to be certain that it was provided by IAP without alteration.
Digital signature verification requires several extra steps, such as retrieving the latest set of Google public keys. You can decide whether your application needs these extra steps based on the risk that someone might be able to turn off or bypass IAP, and the sensitivity of the application.

In Cloud Shell, change to the folder for this step:
```yml
cd ~/user-authentication-with-iap/3-HelloVerifiedUser
```

### Deploy to App Engine
1. Update python runtime to python313.
```yml
sed -i 's/python37/python313/g' app.yaml
```
2. Deploy the app to the App Engine Standard environment for Python:
```yml
gcloud app deploy
```
3. When you are asked if you want to continue, enter Y for yes.
In a few minutes the deployment should complete. While you are waiting you can examine the application files as described below.

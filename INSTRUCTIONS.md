# Social Customer Care [![Build Status](https://travis-ci.org/watson-developer-cloud/social-customer-care.svg?branch=master)](https://travis-ci.org/watson-developer-cloud/social-customer-care)

This application is a Starter Kit (SK) that is designed to get you up and running quickly with a common industry pattern, and to provide information and best practices around Watson services. This application was created to demonstrate how the [Natural Language Classifier][natural-language-classifier] can be used to direct customer requests and queries to the appropriate agent or workflow. Additionally, [Tone Analyzer][tone-analyzer], [Alchemy Language][alchemy-language], and [Personality Insights][personality-insights] demonstrate how to efficiently provide an agent with customer insights.

You can see a version of this app that is already running [here](https://social-customer-care.mybluemix.net/).

**IMPORTANT:**
1. This application requires an AlchemyAPI key with high transaction limits. The free AlchemyAPI key that you request has a limit of 1000 transactions per day, which is insufficient for significant use of this sample application. See [step 3](#step3) of the [Getting Started](#getting-started) section for information about requesting a higher transaction limit on your sample AlchemyAPI key.

2. Running the application will require twitter credentials to access the twitter stream. These credentials can be created at [apps.twitter.com][dev-twitter]. These must be provided in the `.env.js` file.

3. The Natural Language Classifier requires training prior prior to running the application.

## Table of Contents
 - [Getting Started](#getting-started)
 - [Running the application locally](#running-locally)
 - [About the Social Customer Care pattern](#about-the-social-customer-care-pattern)
 - [Adapting/Extending the Starter Kit](#adaptingextending-the-starter-kit)
 - [Troubleshooting](#troubleshooting)

## Getting Started

  The application written in [Node.js](http://nodejs.org/) and uses [npm](https://www.npmjs.com/). Instructions for downloading and installing these are included in the following procedure.

1. Log into GitHub and clone the project repository to a folder on your local system and change to that folder.
2. Create a Bluemix Account. [Sign up][sign_up] in Bluemix, or use an existing account. Watson Beta or Experimental Services are free to use.
3. If it is not already installed on your system, download and install the [Cloud-foundry CLI][cloud_foundry] tool.
4. If it is not already installed on your system, install [Node.js](http://nodejs.org/). Installing Node.js will also install the npm command. Make sure to use node version 4.2.1 as specified in `package.json` or you may run into problems like installation issues.
5. Edit the `manifest.yml` file in the folder and replace `application-name` with a unique name for your copy of the application. The name that you specify determines the application's URL, such as `application-name.mybluemix.net`. The relevant portion of the `manifest.yml` file looks like the following:
```
declared-services:
  natural-language-classifier-service:
    label: natural_language_classifier
    plan: standard
  personality-insights-service:
    label: personality_insights
    plan: tiered
  tone-analyzer-standard:
    label: tone_analyzer
    plan: standard
applications:
  - services:
    - natural-language-classifier-service
    - personality-insights-service
    - tone-analyzer-standard
  name: social-customer-care
  command: npm start
  path: .
  memory: 512M
```
6. Connect to Blumix by running the following commands in a terminal window:
```
$ cf api https://api.ng.bluemix.net
$ cf login -u <your-Bluemix-ID> -p <your-Bluemix-password>
```
7. Create instances of the required Watson services.
  - Create an instance of the [Natural Language Classifier][natural-language-classifier] service by running the following command:
  ```
  $ cf create-service natural_language_classifier standard natural-language-classifier-service
  ```
  **Note:** You will see a message that states "Attention: The plan standard of `service natural_language_classifier` is not free. The instance classifier-service will incur a cost. Contact your administrator if you think this is in error.". The first Natural Language Classifier instance that you create is free under the standard plan, so there will be no change if you only create a single classifier instance for use by this application.

  - If you already have an Alchemy Language service you will use those credentials. Othwerise, create an instance of the [Alchemy Language][alchemy-language] service by running the following command:
  ```
  $ cf create-service alchemy_api free alchemy-language-service
  ```
  **Note:** The free plan of alchemy api has a limit of 1000 API calls a day. This is only sufficient to run quick tests of the starter kit but not fully support the application. The standard plan is not limited w

  - Create an instance of the [Personality Insights][personality-insights] service by running the following command:
  ```
  $ cf create-service personality_insights tiered personality-insights-service
  ```
  **Note:** You will see a message that states "Attention: The plan `tiered` of service `personality_insights` is not free.  The instance `personality-insights-service` will incur a cost.  Contact your administrator if you think this is in error." The first 100 API calls each month are free, so if you remain under this limit there will be no charge.

  - Create an instance of the Tone Analyzer service by running the following command:
  ```
  $ cf create-service tone_analyzer standard tone-analyzer-service
  ```
  **Note:** You will see a message that states "Attention: The plan `standard` of service `tone_analyzer` is not free.  The instance `tone-analyzer-service` will incur a cost.  Contact your administrator if you think this is in error." The first 1000 API calls each month are free, so if you remain under this limit there will be no charge.

8. Create and retrieve service keys to access the Natural Language Classifier
```
cf create-service-key natural-language-classifier-service myKey
cf service-key natural-language-classifier-service myKey
```
9. The Natural Language Classifier requires training prior to using the application. The training data is provided in `data/classifier-training-data.csv`. Adapt the following curl command to train your classifier (replace the username and password with the service credentials of the Natural Language Classifier created in the last step):
```
curl -u "{username}":"{password}" -F training_data=@data/classifier-training-data.csv -F training_metadata="{\"language\":\"en\",\"name\":\"My Classifier\"}" "https://gateway.watsonplatform.net/natural-language-classifier/api/v1/classifiers"
```
10. Create and retrieve service keys for the Alchemy Language service. If you are using an existing alchemy service, use those credentials instead.
```
cf create-service-key alchemy-language-service myKey
cf service-key alchemy-language-service myKey
```
11. Sign up at [apps.twitter.com][dev-twitter] for application credentials. Create a new application with the `Create new app` button and fill out the required form.

12. Provide the credentials to the application by creating a `.env.js` file using this format:
```
module.exports = {
  TWITTER: JSON.stringify([{
    consumer_key: 'twitter_consumer_key',
    consumer_secret: 'twitter_consumer_secret',
    access_token_key: 'twitter_access_token_key',
    access_token_secret: 'twitter_access_token_secret'
  }]),
  TWITTER_TOPIC: '@support',
  CLASSIFIER_ID: ' CLASSIFIER ID',
  ALCHEMY_API_KEY: 'ALCHEMY KEY',
  DEBUG: 'scc:*'
};
```

13. Push the updated application live by running the following command:
```
$ cf push
```

## Requirements

- Nodejs 8 is required
- [Apache Kafka](https://kafka.apache.org/)
- DynamoDB

## Install dependencies

```shell
npm install
```

## Source code lint

eslint is used to lint the javascript source code:

```shell
npm run lint
```

## Configuration

The following config parameters are supported, they are defined in `config/default.js` and can be configured in system environment:


| Name                           | Description                                | Default                          |
| :----------------------------- | :----------------------------------------: | :------------------------------: |
| LOG_LEVEL                      | the log level                              |  debug                           |
| TOPIC                          | the kafka subscribe topic name             |  tc-x-events                    |
| PARTITION                  | the kafka partition            |  0|
| KAFKA_OPTIONS                  | the connection option for kafka            |  see below about KAFKA options                  |
|TC_DEV_ENV| the flag whether to use topcoder development api or production| false|
| TC_AUTHN_URL | the Topcoder authentication url | https://topcoder-dev.auth0.com/oauth/ro |
| TC_AUTHN_REQUEST_BODY | the Topcoder authentication request body. This makes use of some environment variables: `TC_USERNAME`, `TC_PASSWORD`, `TC_CLIENT_ID`, `CLIENT_V2CONNECTION` | see `default.js` |
| TC_AUTHZ_URL | the Topcoder authorization url | https://api.topcoder-dev.com/v3/authorizations |
| NEW_CHALLENGE_TEMPLATE | the body template for new challenge request. You can change the subTrack, reviewTypes, technologies, .. here | see `default.js` |
| NEW_CHALLENGE_DURATION_IN_DAYS | the duration of new challenge | 5 |
| NODE_MAILER_OPTIONS| the node mailer smtp options, see [here](https://nodemailer.com/smtp/ for more detail)| see `default.js` |
|EMAIL_SENDER_ADDRESS| the email sender email address||
|ISSUE_BID_EMAIL_RECEIVER| the email receiver about bid email||
|TC_URL| the base URL of topcoder to get the challenge URL| defaults to `https://www.topcoder-dev.com`|
|GITLAB_API_BASE_URL| the URL for gitlab host| defaults to `https://gitlab.com`|
|PAID_ISSUE_LABEL|the label name for paid, should be one of the label configured in topcoder x ui|'tcx_Paid'|
|FIX_ACCEPTED_ISSUE_LABEL|the label name for fix accepted, should be one of the label configured in topcoder x ui|'tcx_FixAccepted'|
|ASSIGNED_ISSUE_LABEL| the label name for assigned, should be one of the label configured in topcoder x ui| 'tcx_Assigned'|
|OPEN_FOR_PICKUP_ISSUE_LABEL| the label name for open for pickup, should be one of the label configured in topcoder x ui| 'tcx_OpenForPickup'|
|TC_OR_DETAIL_LINK|the link to online review detail of challenge| see `default.js`, OR link for dev environment|
|RETRY_COUNT| the number of times an event should be retried to process| 3|
|RETRY_INTERVAL| the interval at which the event should be retried to process in milliseconds | 120000|
|READY_FOR_REVIEW_ISSUE_LABEL| the label name for ready for review, should be one of the label configured in topcoder x ui|'tcx_ReadyForReview'|
|AWS_ACCESS_KEY_ID | The Amazon certificate key to use when connecting. Use local dynamodb you can set fake value|FAKE_ACCESS_KEY_ID |
|AWS_SECRET_ACCESS_KEY | The Amazon certificate access key to use when connecting. Use local dynamodb you can set fake value|FAKE_SECRET_ACCESS_KEY |
|AWS_REGION | The Amazon certificate region to use when connecting. Use local dynamodb you can set fake value|FAKE_REGION |
|IS_LOCAL | Use Amazon DynamoDB Local or server. |'false' |

KAFKA_OPTIONS should be object as described in https://github.com/oleksiyk/kafka#ssl
For using with SSL, the options should be as
```
 {
    connectionString: '<server>',
    ssl: {
      cert: '<certificate>',
      key:  '<key>'
    }
 }
```

The following config paramaters are supported in the test environment defined in `config/test.js` and can be configured in the system environment. Note that the test config inherits all config options present in the default config and adds/overrides some config options.

| Name | Description | Default |
|:--|:--|:--|
| TC_URL | the topcoder development url |https://www.topcoder-dev.com |
| TC_DEV_API_URL | the topcoder development api url |https://api.topcoder-dev.com/v3|
| MAX_RETRY_COUNT | the maximum number of times to re-test before concluding that test failed | 17 |
| WAIT_TIME | the amount of time in milliseconds to wait before running a re-test | 60000 |
| TC_DIRECT_ID | the topcoder direct id of the repository which is set up with a valid billing account | 7377 |
| TOPCODER_USER_NAME | a valid username for topcoder dev platform | mess |
| HOOK_BASE_URL | the webhook url of topcoder-x-receiver | |
| GITHUB_ACCESS_TOKEN | github personal access token | |
| GITHUB_REPOSITORY_NAME | the name of the repository to create for testing (should not already exist) | |
| GITLAB_USERNAME | gitlab username | |
| GITLAB_PASSWORD | gitlab password | |
| GITLAB_REPOSITORY_NAME | the name of the repository to create for testing (should already exist) | |
| GITLAB_REPO_URL | the URL of the repository to create for testing (should already exist) | |

## Local Deployment

```shell
npm start
```

## Setup for verification
Before verifying the tool, 3 service needs be configured and run them
- processor
- receiver
- Topcoder X

Go to Topcoder X UI login with above used topcoder username and
- go to settings and make sure git hosts are correctly setup, if not click setup and authorize to setup.

- Go to Topcoder X UI and go to project management and add a project from git account and click save, and edit the same project and click 'Add Webhooks' button, verify that webhooks are set up correctly on git host's project.

Now, receiver service can receive the webhooks from git host's project and processor can processes the requests. Now you can verify this service by following the verification steps below

## Run all tests
Configure the Github access tokens, Gitlab username and password in `config/test.js`.

To create a Github personal access token, click on Settings -> Developer settings -> Personal access tokens -> Generate new token -> Provide all permissions.

Configure gitlab username, password, repo name and repo url.

Gitlab Testing:
- Please make sure set repo info via Topcoder-X-UI

Github Testing:
- Do not need set repo info at Topcoder-X-UI


You can then run both github and gitlab tests by using
```
npm test
```

### Run only github tests
```
npm run test:github
```
### Run only gitlab tests
```
npm run test:gitlab
```
## Verification

- create an issue in the repo, you can see the logs in `receiver` and `processor`, the `issue.created` event is generated.

- update an issue in the repo, you can see the logs in `receiver` and `processor`, the `issue.updated` event is generated.

- create a comment on an issue, you can see the logs in `receiver` and `processor`, the `comment.created` event is generated.

- update a comment on an issue, you can see the logs in `receiver` and `processor`, the `comment.updated` event is generated.

- assigned a user to an issue, you can see the logs in `receiver` and `processor`, the `issue.assigned` event is generated.

- unassigned a user to an issue, you can see the logs in `receiver` and `processor`, the `issue.unassigned` event is generated.

- add or remove a label to an issue, you can see the logs in `receiver` and `processor`, the `issue.labelUpdated` event is generated.

- create a pull request, you can see the logs in `receiver` and `processor`, the `pull_request.created` event is generated.

- close a pull request without merge, you can see the logs in `receiver` and `processor`, the `pull_request.closed` event is generated and the `merged` property is `false`.

- merge a pull request, you can see the logs in `receiver` and `processor`, the `pull_request.closed` event is generated and the `merged` property is `true`.

- close an issue in the repo, you can see the logs in `receiver` and `processor`, the `issue.closed` event is generated



### Create a new challenge for a new issue

- Create a new issue in the repo. E.g.

- With the title:  [$1] This is a test issue

- With the body (Markdown supported):

```
This is a description
```

- You will see a new comments

Contest https://www.topcoder-dev.com/challenges/30054075 has been created for this ticket.

```This is an automated message for tonyj via Topcoder X```


- Visit challenge url to verify the challenge:

```
https://www.topcoder-dev.com/challenges/30054075
```

### Update the challenge when the issue's prize was updated

- Update prize of the issue
- With the title: [$2] This is a test issue
- Wait a minute or more
- Visit the challenge url to verify the updated prize and title

### Update the challenge when the issue's title was updated

- Update title of the issue
- With the title: [$2] This is an updated test issue
- Wait a minute or more
- Visit the challenge url to verify the updated title

### Update the challenge when the issue's description was updated

- Update description of the issue
- With the title: This is an updated description
- Wait a minute or more
- Visit the challenge url to verify the updated description

### Update the challenge when assigning the ticket

- Assign issue to member
- Wait a minute or more
- ticket added  [tcx_Assigned](https://github.com/nauhil/test-unit/labels/tcx_Assigned)  and removed  [tcx_OpenForPickup](https://github.com/nauhil/test-unit/labels/tcx_OpenForPickup)
- ticket added assignee
- add comments

Contest https://www.topcoder-dev.com/challenges/30054075 has been updated - it has been assigned to tonyj.<br/><br/>```This is an automated message for tonyj via Topcoder X```

- Visit the challenge url to verify registered member

### Update the challenge when assigning the ticket - no mapping exists

  - Remove exists assignee
  - Remove user mapping data in DB
  - Assign issue to member
- Wait a minute or more
- ticket adds commets

@xxx, please sign-up with Topcoder X tool<br/><br/>```This is an automated message for tonyj via Topcoder X```
- ticket removed their assignment

### Update the challenge when unassigning the ticket

- Unassign member of issue
- Wait a minute or more
- Ticket adds comments
Contest https://www.topcoder-dev.com/challenges/30054075 has been updated - tonyj has been unassigned.<br/><br/>```This is an automated message for tonyj via Topcoder X```
- Ticket updated labels
added  tcx_OpenForPickup and removed  tcx_Assigned labels
- Visit the challenge url to verify no registered member

### Update the challenge when unassigning the ticket - no assigne exists

- Ticket adds tcx_FixAccepted label
- Unassign member of issue
- Close ticket
-  Wait a long time
- THIS TEST WILL FAIL as the issue is not reopened if no assignee exists. It is ignored with a log message

### Update the challenge when close the ticket - without tcx_FixAccepted label

- Remove tcx_FixAccepted label
- Assign issue to member
- Close ticket
-  Wait a minute or more
- Ticket adds comments
This ticket was not processed for payment. If you would like to process it for payment, please reopen it, add the ```tcx_FixAccepted``` label, and then close it again<br/><br/>```This is an automated message for tonyj via Topcoder X```
- Ticket reopend
- Visit the challenge on TC Direct to verfy chalenge status is Active


### Update the challenge when close the ticket

  - Assign issue to member
  - Ticket add tcx_FixAccepted label
  - Close ticket
  - Ticket adds tcx_Paid label
  - Ticket adds comments
  Payment task has been updated: https://software.topcoder-dev.com/review/actions/ViewProjectDetails?pid=30054000<br/><br/>```This is an automated message for tonyj via Topcoder X```
  - Ticket closed comment
  - Wait a long time (Challenge status will change to Active and then change to Completed for long time later)
  - Visit  the challenge on TC Direct to verify challenge status is Active and change to Completed after 17 minute or more

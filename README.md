# cron-tweet

A template repo to schedule tweets via GitHub Actions. Inspired by a recent
conversation with [@GetzlerChem](https://twitter.com/GetzlerChem).

## Requirements

- A Twitter
  [developer account](https://developer.twitter.com/en/docs/twitter-api/getting-started/getting-access-to-the-twitter-api)
  with **Elevated access**. I hope to migrate the tutorial to v2 so that this repo works for basic Essential access.

## Setup

### Create a Standalone App in the Developer portal

Navigate to the Twitter Developer Portal and create a Standalone App. It doesn't
matter what it's called.

![creating a standalone app in the Twitter Developer Portal](https://user-images.githubusercontent.com/24403730/189489479-d0bc66b1-5c0c-4cc7-bef6-4bbc90fcfd5f.png)

### Save API Key and API Key Secret

You will be prompted with your API Key, API Key Secret, and Bearer Token for
your app. To post tweets you will only need the **API Key** and **API Access
Token** Keep these for your records. API keys/secrets are private and should not
be posted or shared publicly.

### Enable read and write app permissions

Beyond you app API key and secret, user-level Access Tokens are required to make
requests on behalf of a specific user. You can generate an Access Token _within_
the Developer Portal for only the Twitter account which owns the app (which is
convenient); however, by default the generated Access Tokens only have **read**
permissions.

In order to enables **read and write** permissions, you will need to extend the
**User authentication settings** for you app in the Developer Portal. There are
several "required" sections within the **User authentication settings** panel,
but you only _really_ need to select "Read and Write" in the **App
permissions**.

For **Type of App** select "Web App, Automated App or Bot". Since we are only
generating an Access Token _within_ the Developer Portal we don't need a valid
Callback URI either. Just put the link to your GitHub repo for the Callback URI
and Website URL.

<p align="center">
<img width="400" alt="user authentication settings form for the app" src="https://user-images.githubusercontent.com/24403730/189490431-bffd406b-5ce5-452b-bf3f-12b6a8d44872.png">
</p>

You will be prompted with Client ID and Client Secret. Honestly I don't know
what these are, but you won't need them in this tutorial.

### Create an Access Token and Secret

Navigate to the "Keys and tokens" panel for your app in the Developer Portal and
click "Generate" under **Authentication Tokens** to generate an Access Token and
Secret for your Twitter account. Save these tokens for your records.

Make sure the "Access Token and Secret" section now says "Created with Read and
Write permissions".

<p align="center">
<img width="400" alt="Keys and tokens panel within the Developer Portal with valid Access Token and Secret" src="https://user-images.githubusercontent.com/24403730/189490761-b8c99353-4cb6-4e6e-b2bf-429825c21c4b.png">
</p>

You should now have the following saved for your records:

- API Key (consumer key)
- API Key Secret (consumer key secret)
- Access Token
- Access Token Secret

You can create a `.env` text file for your records with the following contents,

```
TWITTER_API_KEY=<API Key>
TWITTER_API_SECRET=<API Key Secret>
TWITTER_ACCESS_TOKEN=<Acesss Token>
TWITTER_ACCESS_TOKEN_SECRET=<Access Token Secret>
```

This file is ignored by `.gitingore` so you won't accidentally check-in your
private keys to your public repository.

## Getting started

Now you should have all the credentials you need to tweet on behalf of your
authorized account! There are several different Twitter clients for making
authenticated requests with these credentials. This repo uses
[Twurl](https://github.com/twitter/twurl), a simple Ruby command line
application.

### Your first tweet

Make sure you have `twurl` (and Ruby) installed on your machine
(`gem install twurl`), and you're all set to make requests.

The following command will post a status update of "Testing 1, 2, 3, ..." to
your account.

```bash
source .env # add your keys/secrets to your shell environment
twurl request \
	--consumer-key $TWITTER_API_KEY \
	--consumer-secret $TWITTER_API_SECRET \
	--access-token $ACCESS_TOKEN \
	--token-secret $ACCESS_TOKEN_SECRET \
	--data 'status=Testing 1, 2, 3, ...' \
	/1.1/statuses/update.json
```

### Automating tweets (this repo)

The 🤖 "bot" for this repo is defined in `.github/workflows/tweet.yml`. The
`.github` directory in this repo is special folder which GitHub inspects for
various files. Files within `.github/workflows/` are custom YAML configurations
which define automated processes for GitHub to trigger on various
[event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows).

Fortunately, there is a
[`schedule`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)
event which allows us to write a "workflow" to schedule our `twurl` usage.

> **Note** Other events like
> [`push`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push)
> are commonly used to trigger a workflow when changes are made to a specific
> branch.

The workflow included in this repo does the following every day (at 9AM UTC),

- Setup a Linux machine
- Installs Ruby
- Installs `twurl`
- Tweets with `twurl`

### ⚠️ IMPORTANT ⚠️ Adding your secrets to the GitHub Repo

Without your credentials, the 🤖 will fail to tweet on your behalf 😢.
Fortunately, there is a safe way to expose your secret credentials to GitHub
Actions (without exposing your keys publicly).

Within the "Settings" tab for this repo, select Security > Secrets > Actions and
click "New repository secret" to add each of your keys/secrets to the repo.

Use the naming convention from the `.env` file (e.g., `TWITTER_API_KEY`) for
each of your secrets. These names are referenced withing the workflow file, so
using different names or misspelling will cause the tweeting to fail.

![Adding secrets to repo setting](https://user-images.githubusercontent.com/24403730/189492790-fd1f28f5-39dd-4819-a098-92455418ddc1.png)

That's it! Your repo is all setup to tweet on your behalf.

### Be creative 🎨!

This repo is strictly for fun meant to provide minimal scaffold to play around
with the Twitter API via GitHub Actions. You get 2000 min of GitHub Actions per
month for free, so use it as a playground to try out different things:

- Schedule tweets
  [multiple times per day](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)
  with different contents depending on the time of year or day
- Tweet when someone
  [opens or closes an issue](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#issues)
  in this repository

# Apple shortcut for Firefly III

#### A tutorial to automatically add expenses to your self hosted version of [Firefly III](https://github.com/firefly-iii/firefly-iii)
---

## Prerequisites

This tutorial assumes you have a working self-hosted instance of Firefly III that can be reached from the internet. If you haven't already, refer to their installation guide. I suggest [using Docker](https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker/). Personal note, be careful about the password your try to input when creating the first account. It must be insanely long and I first thought something was wrong during the installation.

What you'll need:
- Your API URL. The endpoint we will use consists of the URL + /api/v1/transactions. Thus, it must look something like: https://yourfireflyiiiurl.com/api/v1/transactions
- Your personal access token. You can see it and copy it somewhere safe only once, when you create your profile. If you don't have it, you must create a new one. From your dashboard, go to Options>Remote access and tokens. Remember that this value will remain visible in your shortcut, this could pose security risks.

## Let's begin: the amount and the merchant values

First thing to know, before iOs 26, Shortcut would allow you to run a shortcut from an automation. Afaik, this is not possible anymore, because in the first step you'd get [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/1.png?raw=true). You must run all the actions from the automation instead. 

So, let's create a new automation. Shortcut>Automation>"+" button at the top right>Wallet "When I tap a Wallet Card or Pass">Select all the cards you want this automation to work with. 

I selected "Run immediately" but you may want to try "Run after confirmation," we'll see later why.

Select "Create New Shortcut"

First thing we need to look for a text action. "Search Actions">Text. Tapping on the text block, let's click on "Select variable" and let's tap on "Shortcut input". [This](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/2.jpg?raw=true) is what it should look like. Tap on the voice "Shortcut input" and select "Amount". It should now look like [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/3.jpg?raw=true).

We now need to manipulate this amount value. The format from Shortcut is "10,09 €"; Firefly wants only "10.09". So, "Search Actions">Replace text. Make the action look like `Replace '€' with '' in 'Text'`. Then, look for the replace text action again. This time, make it look like `Replace ',' with '.' in 'Updated text'`. Let's now call this manipulated string something memorable. "Search Actions">Set variable. You can use the name you want, I went for `Set variable 'formatted amount' to 'Updated text'`. Of course this time, the 'Updated text' must be the second one. All in all, everything should look like [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/4.jpg?raw=true).

Let's now create a text version of the merchant. "Search Actions">Text. Inside the text block, tap "Select Variable">Shortcut input. Let's then tap on Shortcut input and select "Merchant". I wanted to call this text "description", so I did "Search Actions">Set variable and I made it look like `Set variable 'description' to 'Text'`. You may want to name it otherwise, feel free to do it.

## The categories

Firefly III allows us to categorise the expense, so in this section we can create a list of categories we may use. "Search Actions">List. Create the list as you like. Once you created the options you prefer, "Search Actions">Choose from list, and then "Search Actions">Set variable(so that this choice has always the same name. In my case, everything looks like [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/5.jpg?raw=true).

This is the moment when you may want the automation to run only after a confirmation. If the automation runs immediately, and the list appears waiting for your input, it may happen that in the meantime the screen goes off, or that you accidentally click the side button. In these cases, the list disappears and the automation silently fails. 

##  The API request

Let's have a look at an API transaction request to Firefly III

<details>
  <summary>What it looks like</summary>
  
  ```
  POST /api/v1/transactions HTTP/1.1  
Host: your-firefly-url
Content-Type: application/json
Authorization: Bearer YOUR_ACCESS_TOKEN
Accept: application/json

{
    "error_if_duplicate_hash": false,
    "fire_webhooks": true,
    "apply_rules": false,
    "transactions": [
      {
        "type": "withdrawal",
        "date": "2026-09-05",
        "amount": "10",
        "description": "Coffee shop",
        "source_id": "1",
        "destination_name": "Some Café",
        "currency_code": "EUR"
      }
    ]
  }
  ```

</details>



We will first focus on the body of the request.

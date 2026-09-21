# Apple shortcut for Firefly III

#### A tutorial to automatically add expenses to your self hosted version of [Firefly III](https://github.com/firefly-iii/firefly-iii)
---

## Prerequisites

This tutorial assumes you have a working self-hosted instance of Firefly III that can be reached from the internet. If you haven't already, refer to their installation guide. I suggest [using Docker](https://docs.firefly-iii.org/how-to/firefly-iii/installation/docker/). Personal note, be careful about the password your try to input when creating the first account. It must be very long and I first thought something was wrong during the installation.

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

### What a request looks like

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

### The body of the request

We will first focus on the body of the request. What do 'fire_webhooks' and 'apply_rules' do? I don't know 🤷. 'error_if_duplicate_hash' allows you to block duplicate expenses. I think you can remove these voices if you want, but try at your own risk. 

Let's create a dictionary (the first `{}` of the body) to build the body of the request. "Search Actions">Dictionary. For the first three entries, select Boolean and set the value you want. **Watch out for the transaction value**. It may look like you need a dictionary, but what you really need is an array. This is because the structure of the transactions value is not `"transaction": {}`, but rather `"transactions": [{}]`. The array is the same for the square brackets, the dictionary inside the array for the curly brackets. 

Then, in the first item of the array you create a dictionary. [This](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/6.jpg?raw=true) is how the dictionary looks. Then, inside the transactions value, you must see [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/7.jpg?raw=true) after you created the first item(which, remember, must be another dictionary). It's important you read an unchangeable 'Item 1' on the left, because that means that the voice you used for the transactions value is an array. So, "Item 1" must be a dictionary, inside this every value must be text.

- type: withdrawal
- date: from the bottom bar select "Current Date". **Important**, click on "Current Date" and select "ISO 8601" as "Date Format." You can toggle "ISO 8601 Time" if you want(I recommend it). Firefly III does not accept other formats, afaik.
- amount: "Select variable">select 'formatted amount' or the name you have chosen for the manipulated string of the amount.
- description: "Select variable"> select 'description' or whatever you named the text block containing the merchant value.
- source_id: 1 (this means that the money will be deducted from the first account you set up in Firefly III. You may want to change this value; to see what number you need, go to your dashboard>accounts>asset accounts>click on the account you want to take the money from and look at the number at the end of the URL).
- destination: "Select variable"> select 'description' or whatever you named the text block containing the merchant value (or just don't use this value).
- currency_code: EUR (of course, if you pay in US Dollars write USD etc.; three-letters codes only).
- category_name: "Select variable"> select the variable you used to define the choice from the category list (in my case, "category").

[This is how it should look like](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/8.jpg?raw=true) remember, all these entries must be text.

### The headers of the request

Let's now move to the creating the headers of the request and the request itself. "Search Actions">Get contents of URL. The url must of course be API endpoint: https://yourfireflyiiiurl.com/api/v1/transactions. Open the entry by tapping on the ">" button, change the method to POST. Tap on add new header three times. These must be the values.

- Accept: application/json
- Authorization: Bearer YOUR_ACCESS_TOKEN (here, replace YOUR_ACCESS_TOKEN with your actual token...copy and then just copy it once, and don't touch this entry again as this token is insanely long)
- Content-Type: application/json

when the entry asks the request body, tap on "File". When it asks for the file, select the dictionary we created in the section before. It should look like [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/9.jpg?raw=true)

### This is (not) the end: the response

In theory, you don't need anything else; at every tap with a POS, after a while (the time apple wallet shows the notification) you'll see the list of categories. Choose one and the automation will be executed. However, looking at [the response from the API framework](https://api-docs.firefly-iii.org/#/transactions/storeTransaction), I noticed the voice "source_balance_after". The framework can tell you how much remains in your account after the purchase. This is a feature that I've found in NO bank apps but Revolut and I've always found very useful. I guess that for banks what's important is that you spend money, not that you save it. Of course, you can skip this part if you're not interested.

To fetch it, first thing you must create a dictionary from the response of the API request. So "Search Actions">Get dictionary from input result. And connect the input result to the "Get contents of URL" action: like [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/10.jpg?raw=true).

The voice is inside data>attributes>transactions. So then we will need to access these values first, and then the voice we're looking for. It's a [4-actions-step](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/11.jpg?raw=true).

Once we have this value, we can just create a Text block and paste it on it. I personally used an If condition so that, if for some reasons the value is empty, I can read the whole response, which will probably report an error. It should look like [this](https://github.com/Namekinek0/Apple-shortcut-fireflyiii-tutorial/blob/main/img/12.jpg?raw=true).

That's it, happy expense-tracking! ✨

# Send unlimited Whatsapp messages from Python - for free!

It's true - you really can send free messages to five (verified) phone numbers for free using Meta's [WhatsApp Business Cloud API](https://developers.facebook.com/products/whatsapp/).  The only proviso is that you agree to and follow their terms and conditions, especially regarding opting-in and acceptable use.

The even better news is that I've waded my way through the API docs and spent time going down a few rat-holes so you won't have to! The result is a bare-bones wrapper in about 20 lines of Python which you can copy into your own scripts to enjoy outbound Whatsapp notifications without having to learn the quirks of yet another bloated library/API yourself.  Or if you _do_ want to get a deeper understanding, hopefully this will fast-track your journey and move you towards the advanced features quicker.

### TLDR: Here are the essential 20 (ish) lines of code in Python:

![](Screenshot%201.png)


You'll need to jump through a few (easy) hoops to get set up with Meta before you can acutally send messages, but once you're done it'll be as easy as calling:

```

>>> Whatsapp()

# Sends a test message with test url (and preview) to your default contact

>>> Whatsapp("Hello Monty", CONTACT[4])

# Sends "Hello Monty" to your fifth contact

>>> wa = Whatsapp(autosend=False)

# Create but don't send a message.  Useful when testing/debugging if you don't want to actually spam someone, and also if you intend to do any pre-processing to further refine the message before finalising, or post-processing e.g. to save the message data somewhere, or if you're making a 'factory' to create multiple messages and send them later, perhaps according to a schedule or in response to certain events.

>>> wa.send()

# Send (or resend) a previously prepared message

```

### And here's the config file you'll need to populate once you've created an App on the Facebook developers' portal:

![](Screenshot%202.png)


> *SOAP BOX alert! For some reason almost every tutorial about APIs seems to mix credentials and config data in with the main script.  This is officially, metaphysically, didactically and canonically **bad practise** and not to be emulated, so let's start the right way from the outset!*

You can find tests and the latest version of the code to copy/paste/download/fork on my [Github repository](https://github.com/PFython/Whatsapp).  Please remember to give it a Star when you visit, and if you want to pay it forward, maybe buy me a coffee?

<a href="https://www.buymeacoffee.com/pfython" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-yellow.png" alt="Buy Me A Coffee" width="217px" ></a>

Also if you want to use the full range of message types (audio, contacts, documents, images, templates, videos, stickers, locations, and interactive) and don't mind an extra dependency in your code you can import the whole package from PyPI:

```
python -m pip install whatsapp --upgrade
```
> While the
____

# More details... if you're still reading!

## Getting Set Up

The WhatsApp Business Platform is intended _"for people developing for themselves or their organization, not on behalf of a client"_ and _"the platform is not open to those developing on behalf of clients."_ ... but don't let that put off!

In fact compared with getting up to speed with Google APIs and authentication, the Meta/Facebook documentation was so clear and easy to follow that I don't plan to walk you through it.  Just follow Steps 1 and 2 of the ['Getting Started'](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started) instructions.  You can go further if you want to learn how to receive inbound messages using Webhooks and making things interactive, but this article is all about providing 80% of the useful functionality for 20% of the effort, and if you're at all like me, you'll be tremendously excited just at the prospect of being able to send free notifications from your Python code to your phone (and four others).

Having gone through the Getting Started instructions, my only other comments before you go there yourself are:

- You don't actually have to create or verify a Facebook Business Account.  At a certain point when you're creating your new App you'll be given the option of creatring a Test Business Account.  That's all you need.

- You don't _have_ to even enable two factor authentication if you just want to dive in and start playing.

- Once you've created your five free contact phone numbers I don't think there's a way to change or delete them... so choose carefully and triple check they're correct and properly formatted before you commit.

- As well as your **Token**, you'll just need to copy/paste your **Phone number ID** and **App ID** into `config.py`. The phone number ID is different from your actual (test) phone number, which the code doesn't need.

   > **NB** *For Test Business Accounts, the Token expires every 24 hours so you'll either need to update your `config.py` manually or write a bit of code e.g. with `selenium` to log in to your Facebook Developer Dashboard and refresh and copy the Token.  I've included `APP_ID` and `APP_URL` in `config.py` for that purpose, and if you add your scraping code into `config.py` it should run and update automatically whenever your credentials are imported.*

- The official way of initiating messages to contacts who have opted in to receive them is via the Template message type which is why the first test message in the documentation (sent by CURL) uses a predefined template called `hello_world`.  The only trouble with this is that you need to set up new Templates and _have them approved_ via the Cloud API dashboard, every time you want a new form of message.  Presumably if you're reading this you're already thinking about generating messages and content yourself in Python, not necessarily fiddling around setting up server-side templates and passing variables into _them_ every time.  So a simpler starting point for understanding the API as a whole, and the basis for my `Whatsapp` class above is the simpler 'Send Messages' API which you can read more about [here](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages).

- As well as verifying each phone number before being able to send to it, you'll need to "Allow" the test account to send to each phone when you receive the first "hello_world" test message.  That should open the gates to sending more Template messages but you'll probably prefer is to continue the conversation sending non-Template messages - especially if you're doing something like sending notifications and updates to yourself and team programmatically.  In that case you'llD you'll need to actually **send a reply** to the Test Business number, or in other words, initiate the conversation.  It took me ages to work out why I was getting success (200) messages from my API calls but not receiving anything on my receiving phone(s).  This was the reason, but the documentation doesn't mention it!  It also looks like conversations reset themselves every 24 hours, so you'll either need to use Templates after all, or keep the conversation going from the receiver's side every day.  One option for doing this is using the free Business version of the Whatsapp phone app and creating an auto-reply.

- If you want to unlock the full power fo the Whatsapp Cloud API and remove all restrictions, you'll need to create or link to a verified Business Account.  If you do so you currently get 1,000 'conversations' free per month or in other words roughly 33 messages per day - one an hour plus a few spare.  But unless you do something to get kicked off your Test Business Account, _unlimited_ free messages to/from five phone numbers is very generous and fantastic for developers and micro-businesses.

## The Code

- If you're lucky you'll be more familiar with CURL than I was when wading through the [Developer docs](https://developers.facebook.com/docs/whatsapp/cloud-api).  That's one thing the Google API docs do better than Meta/Facebook... they give sample Python code.  But never fear, you can use [tools like this](https://reqbin.com/req/python/c-xgafmluu/convert-curl-to-python-requests) to convert their CURL examples into Python if you want to learn more and dig deeper.

- My `Whatsapp` class is intentionally minimalistic i.e. just enough functionality to get you started and send a basic text message with a url and preview.  The whole point of the `kwargs` logic however is to make it [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and easily extensible e.g. if you want to specify more elaborate message types like audio, video, images, buttons, geolocation, and yes, templates if you must.  To do so, you would just provide a new `data` dictionary or keyword arguments.  If that sounds too abstract or you're not familiar with `*args` and `**kwargs`, have a look at `test.py` which I hope will give you some ideas.

- Not essential, but `self.data` uses my go-to helper class [`CleverDict`](https://github.com/PFython/cleverdict).  It basically adds the ability to get and set dictionary values using the oh-so-convenient 'dot' notation e.g. `data.type` rather than `data["type"]` which is one keystroke for "." rather than five - yay!  You can easily strip it out if you prefer to be completely dependency free, but `cleverdict` is very lightweight and worth checking out if you haven't used before.  I've used it extensively in `tests.py` so if you do strip it out, you may need to refactor those tests too.

## And finally...

I've always been a fan of "simple tools for busy people", so `whatsapp.py` and `config.py` are intentionally short and offered "bare-bones" for copy/pasting.  I don't doubt some excellent Python modules will come to the fore which wrap the Whatsapp Cloud API more comprehensively, but in the meantime if you feel the urge to build on this humble repository and want to suggest Pull Requests I'd be happy to collaborate and perhaps get this published on PyPI.  No point worrying about docstrings, typing, `__repr__` etc until then!

If you want to get in contact, the best ways for me are either through [Github](https://github.com/PFython) or [Twitter](https://twitter.com/AWSOM_solutions).  Please note that I do work full time, I have a very needy dog, and also act as a carer, so please be patient if I don't respond quickly.  Enjoy.

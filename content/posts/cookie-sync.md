+++
title = "Cookie sync in programmatic advertising"
categories = ["programmatic advertising"]
tags = ["cookie", "RTB"]
date = "2017-03-29T11:35:33+08:00"

+++

Cookie is a browser mechanism to store small piece of information in client’s browser. Now days, cookie is widely used in:

	- Sign in. Store session in cookie for server side authentication.
	- User preference. It stores user preference and load them when open the site which makes the UI user friendly.
	- eCommerce. Remember the good in your shopping cart.
	- Tracking behavior.  It records the user behavior for better targeting.
	- Advertising. It carries various information through couple of counter parties in advertising realm. DSP, SSP, Ad Exchange, etc.

Though cookie is prevalent, for security reason, cross domain policy restricts cookie only in the domain which cookie is set. A cookie set by *foo.com* is not allowed to be accessed by *bar.com*. (Cookies do not provide isolation by port). In advertising, cookie might have user ID and information stored which needs to be shared among individual parties with different domains. How could that be achieved? 

Introducing cookie sync (or cookie mapping).

First we will elaborate how cookie sync works with a real life scenario, then we track the data flow, finally you will find a runnable code example in our github repo and you can read through and run by yourself.

### Scenario - retargeting users

A demand book selling site want to retarget users who did browse the book but finally left without conducting a payment. A user might leave for many reasons, but as long as the user shows interest in a particular book, the company would like to retarget the user at a later time when the user intends to buy books again. 

Demand site asks DSP for help to achieve that. Once a user browses a book, a pixel will be triggered to DSP. DSP generates an unique ID for that user (anonymous) and keeps in database. DSP also sets cookie in client side, so that next time when the user come back(piggyback), DSP should know. But DSP alone is not enough, because the book company wants the user to see the book when he is browsing another publisher sites.  DSP will bid for inventory to show the book ad, but at what price is most effective? Obviously, if DSP knows who is the user browsing that inventory then it can make wise bid strategy.  So DSP needs to sync information with SSP (who requests the inventory auction)  by cookie. 

If you still remember, DSP sets a cookie for user (DSPID), it also responses with a pixel to be triggered to SSP site carrying the DSPID in URL. Now SSP knows the user and the DSPID of the user. SSP generates an SSPID and stored it together with DSPID in database(it’s not a MUST), and then set SSP cookie to user browser. After that, SSP redirect back to DSP with SSPID attached.  So that DSP knows the mapping of DSPID/SSPID. When a ad request is issued from publisher site, SSP will call for auction to every DSP with  SSPID carried as well as the ad meta. When DSP received the auction call from SSP, if it sees the SSPID and find in database, then DSP bid for the inventory, otherwise DSP should conduct a negative bid strategy. If DSP wins, the user will see the same book ad in publisher site.

That’s the basic idea of RTB and cookie sync’s role in between.

### Flow

![cookie-sync](/images/cookie-sync.jpeg)

	1. User browse the book in demand site which have a pixel embedded
	2. A pixel is triggered to DSP. DSP save the DSPID in database
	3. DSP set the cookie back to browser and send back a pixel (SSP site)
	4. Pixel for SSP, carring DSPID in URL. Start cookie sync
	5. Redirect back from SSP with SSPID, cookie is sync
	6. DSP now knows the SSPID and update database for DSPID
	7. User browser publisher site at a later time
	8. Ad tag requests SSP to get an ad to display
	9. SSP request an ad auction to every DSP, SSPID is carried in request body as well as ad meta
	10. DSP receives the auction request and find the SSPID in database. DSP bids for inventory
	11. SSP decides which DSP wins and response with ad information. If our DSP wins, user will see the book again.

### Code example

You can find the runnable code demo [here](https://github.com/implustech/cookie-sync-demo)
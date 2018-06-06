---
layout: post
title:  A look at the current state of dApp tooling
category: Web3, JavaScript, React, Python
description: The current state of dApp tooling, from a bootstrapped founder’s perspective.
---


<!--description-->


Six months ago, I decided to build a dApp startup that didn’t involve an ICO. I found the idea of building a decentralized application fascinating, but also I found the whole ICO environment pretty slimy/toxic. My goal was to create a product — not a gigantic windfall of hype and tokens. So I set out with a friend of mine to create an entirely bootstrapped startup that uses an Ethereum escrow smart contract for payments (you can check out the current state of the app at [dimpull.com](dimpull.com). After building out the basics of the application, I quit my soul-sucking software developer role at Bank of America to pursue this full time. That was just a few months ago. Now that the app is live, I’d like to share my completely biased opinion of the current state of dApp tooling, as well as the general developer experience, from my perspective as a full-stack web developer and startup founder.

![]({{site.baseurl}}/assets/img/web3.jpg)

<br>

To build Dimpull, most of my time has been spent building a web app the way web developers are used to — a React frontend, a Python/Flask backend, and a Postgres database. Those are not blockchain technologies, and to a large extent, Dimpull is not a dApp. But instead of outsourcing the payment system (hooking into Stripe’s API and holding onto customers’ money as a middle-man), I set out to build an [escrow](https://en.wikipedia.org/wiki/Escrow) system for transferring value on the Ethereum blockchain. And that has been the most exciting and interesting part of the development experience so far.

This isn’t a tutorial. This is a record of my experiences in this new world of dApp development, in the first half of 2018…

<br>

### Writing a smart contract:

This was pretty painless, and quite fun. There are some nice tutorials available for Solidity, which basically feels like writing statically typed Javascript (well, it’s a bit clunkier than that, but not so bad). [Coursetro](https://coursetro.com/courses/20/Developing-Ethereum-Smart-Contracts-for-Beginners) has a nice short course for getting started.
Once I had something that seemed to work, I posted my contract to /r/ethdev/ and quickly got useful tips from other developers. The community is highly active and helpful. Solidity = not so bad! **4 Stars**.

<br>

### Testing a smart contract (the nascency of this space starts to show):

I started trying to test the functions in my contract on the fantastic online [Remix IDE](https://remix.ethereum.org/), while running a TestRPC on my machine. And I couldn’t do it. For some reason, I’d run out of gas every time I called the transfer function. [Truffle Framework’s](http://truffleframework.com/) personal Ethereum blockchain, [Ganache](http://truffleframework.com/ganache/), could not support `.transfer()` in my app without throwing an `out of gas` error, which seems like a glaring bug for a blockchain made to run tests! Once I realized this, I used Remix’s Javascript VM and thanked the blockchain gods that my contract worked the way I expected.

*So testing is broken?*

Nope! A newer version of Ganache resolved my headache. I should mention that it’s a great tool when it works: watching a blockchain run on your local machine is quite gratifying. And writing [smart contract tests in Javascript](http://truffleframework.com/docs/getting_started/javascript-tests) feels like writing tests for a web app — couldn’t be much more intuitive than that… **3.5 Stars**.

<br>

### Integrating with web3 (MetaMask):

[MetaMask](https://metamask.io/) is brilliant, and it doesn’t take much time to figure out the basics. Props to the MetaMask team for making interacting with “the distributed web of tomorrow” quite easy. Seeing the extension pop up from my client’s request filled me with joy (I’m not kidding). Stuff like this gives me a lot of hope for whatever the distributed web of tomorrow will be. **5 Stars**.

![]({{site.baseurl}}/assets/img/metamask.png)

<br>

### Using web3.py and py-solc:

Dimpull can’t just interact with the blockchain from the client side. It also needs to be able to call the smart contract from the server, to release funds from the escrow. It seems that most people interacting with web3 are doing it from the client side, so the web3.py community appears to be quite a bit smaller than web3.js.
Outside of truffle-land, you need to install the solidity compiler separately, which for some reason isn’t as easy as a pip install. WHY ISN’T INSTALLING PY-SOLC AS EASY AS A PIP INSTALL? And I ran into a show-stopping issue where I couldn’t use web3.py because I couldn’t compile my smart contract in python.

![]({{site.baseurl}}/assets/img/pyeth.png)


Fortunately, the gitter community is very active (this is definitely a theme in the dApp world!) And since the community is so small, I had an extended troubleshooting session with the maintainer of the py-solc repo (thanks @pipermerriam)! The solution to my issue was to downgrade my py-solc version from 0.4.19 to 0.4.17… which is not a good sign. ¯\_(ツ)_/¯ But it worked, and so on I went. **2.5 Stars**.

<br>

### Deploying to Test Networks:

I was nervous about this part. It seems that getting my smart contract onto an actual distributed blockchain would be a slow and painstaking process… But it wasn’t. Truffle was again a savior by making this painless, when integrating with [Infura](https://infura.io/). And running the dApp from the Ropstem TestNet is another moment where I realized that this is really cool. My test transactions are actually being mined, and it’s out there on a blockchain for anyone to play with? That’s cool! MetaMask continued to be an intuitive and slick interface; switching between blockchains is easy as pie.

The TestNets aren’t all smooth sailing though. Once your application is on a TestNet, the debugging tools are pretty much non-existent. [Etherscan](https://etherscan.io/) lets you check if a transaction failed or passed… which is nice, but if it fails, there isn’t much you can do to figure out why. I deployed my contract to Ropsten and Rinkeby, and it when it didn’t work on Rinkeby, it was essentially impossible to figure out why. **4.5 Stars**.

<br>

### Deploying to the Main Ethereum Blockchain:

This should have been easy. After all, I’d already deployed the contract to Ropsten and Rinkeby, so I just needed to do the same process: [use Truffle to deploy the contract via an Infura node](http://truffleframework.com/tutorials/using-infura-custom-provider) (I didn’t want to run a full node myself). However, I couldn’t do it. I kept wasting gas (about $2.50) trying to use Infura, and receiving various Infura errors, such as `nonce too low`. I ended up trying to deploy via the online IDE, Remix, and was able to do that with no issues. Infura is a brilliant tool — but issues like this are disheartening.

But Dimpull’s escrow contract was successfully deployed, so we were ready to accept payments. I wanted this to be 5 stars so badly, but Infura’s instability took it down a notch: **4 Stars**.

<br>


## Overall: 24.5/30 stars: ¯\_(ツ)_/¯ (good enough)

### Tools I’m looking forward to using:

I’m not a purescript developer (yet), but it’s not hard to see why stronger types in the Ethereum Stack would make for a more secure development process. I had the pleasure of meeting Martin Allen who is working on [purescript-web3](https://blog.foam.space/purescript-web3-release-631b16bec7a) and [chanterelle](https://github.com/f-o-a-m/chanterelle) as an alternative to Truffle. I plan to give these a try in the near future. I really like the control chanterelle gives for configuring deployments.

I’ve also started messing around with [cliquebait](https://github.com/f-o-a-m/cliquebait), which is a docker image that spins up a geth node, which is a fantastic alternative to TestRPC. This should be in every Ethereum developers toolkit!

<br>

### Is web3 ready for startups?

I discussed the major tools and systems I needed to use, roughly in the order that I encountered them for building Dimpull. There were frustrations along the way, but there were also moments of joy. Honestly. These tools are all brand new, and in various stages of beta. And they are all aiming at making it easier to get to those moments of joy. The whole system really is magical, and every time you get something working, you feel like you are interacting with something strange, powerful, and exciting. I can’t compare it to learning any other technology from a web developer’s stack, you really should build and deploy something to feel it for yourself. And you should start now. The tools are good enough, some are great, and there is so much being improved upon everyday that this post will probably be a historical artifact in 6 months.

<br> 

### What’s the point of all this?

Most of Dimpull is built with ‘traditional’ web technologies. But cutting out the need to connect to a payment processor gives more power to me as a developer. **I didn’t have to connect to a 3rd party API to add the ability to transfer value**. In the future, you can dis-intermediate EVERY part, not just value transfer, to make unstoppable, truly decentralized applications, that will *flip everything you know about your current web development stack on it’s head*. Hopefully this gave you a taste of what you can expect for now.

![]({{site.baseurl}}/assets/img/tbt.png)


If you are interested in this project, check it out at [dimpull.com](dimpull.com). Dimpull is a platform for people to schedule 30 minute phone calls with experts in the blockchain space. Payments are backed by and escrow smart-contract on the Ethereum Blockchain, which is pretty cool.






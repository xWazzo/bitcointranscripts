---
title: Static Invoices in Lightning: AMP/Offers/LNURL
transcript_by: adamjonas via TBTBTC v1.0.0
media: https://www.youtube.com/watch?v=WsSa00gUvZw
tags: ["lightning","Bolt 11","Bolt 12","invoice","LNURL","AMP"]
speakers: ["Elle Mouton"]
categories: ["conference"]
date: 2021-11-13
---

And the slide is kind of given away already.

So please welcome Elle Mouton from Lightning Labs.

Hello, hello. Is this working?

Okay, cool, awesome.

Hi guys, I'm Elle, and I'm going to be talking about static invoices in the Lightning Network.

So as most of you probably know, in Lightning, currently if you want to be paid, you have to create a new invoice every time. And this is a bit of a difficult thing for people to understand who don't know anything about the space.

So today I'm going to just be diving into exactly why it's a mission for us to have these static invoices, and then just outlining the three various proposals LNURL offers an AMP that people have been discussing.

Before I carry on, I just want to mention a few things. 

So first of all, if you're a Lightning developer, this talk is really not for you. Like it's for, I just want more people to be able to join the discussion. So it's just making sure everybody understands the various components, right?

So yeah, to the Lightning developers, if I say anything incorrect, please shout at me.

And then another thing is, the one thing that all these three proposals have in common is they give us these static invoices, but each of them have other features that are really cool, and I will mention those as well. We can't really compare them.

Okay, just a quick talk overview.

So I feel like it's very important to make sure everybody understands what the issue is, so I'm going to spend a little bit of time on a payment deep dive. Not a deep dive, just an overview so everyone understands what we need to have payments work in the Lightning network.

From that, we'll understand why bulk invoices are a thing.

Then I'll talk about the issues with those things, and then from there we can go into the various proposals.

And then I'll just give a quick high level overview and mention some of the arguments that people have been talking about on Twitter.

And leave you with some questions to think about.

Okay, so as all of you probably know, Lightning is made up of a bunch of channels, which is just a contract between Alice and Bob, they've got a two of two multi-sig, allows them to do value transfer between each other many times, very fast, off-chain.

So really cool, if the whole Lightning network is made up of channel between every person, static invoices are solved, not a problem.

I can finish my talk right now.

But the thing is, this doesn't scale, right?

You can't have a UTXO open with every single person in the world that you're gonna wanna have to pay, so this doesn't work.

And the beauty of Lightning network is that Alice's view can look a little bit more something like this, and all she needs to do to pay someone like Dave is find a route to Dave. 

Cool, so let's look at that a bit. 
So she finds a route to Dave, and she just needs to focus on that. 

Cool, so now how does Alice pay Dave, right? Why can't this be static? Why can't she just tell her node, please pay Dave's node ID this amount of Bitcoin. That's a static thing, right? So the naive way would be Alice just tells Bob, hey Bob, please help me out. I'm gonna tip you, I'm gonna leave one Bitcoin with you, and please just push two over to Charlie, and tell him to do the same with Dave. 

Cool, that's great, Dave got paid. That doesn't work, because in Bitcoin we can't trust Bob, right? So in reality, if Alice pushes money over to Bob, there's nothing stopping Bob from just running away at this point. 

Okay, so Lightning payments don't work like that. How do they work? What we actually do is, before the payment can happen, before we even touch the Lightning network, Alice says, hey Dave, I'm gonna pay you, please be aware. Dave goes ahead and generates a secret, takes the hash of that secret, and sends it back to Alice. 

This happens all outside of the Lightning network. 

Okay, now Alice goes again to Bob, but this time she tells him, hey Bob, you can have these three Bitcoin, but only if you can unlock this hash. If you can't unlock it, I'm gonna get the money back after some time lock. 

So Bob sees this, realizes he's only gonna get a tip if he continues the pattern onto Charlie so that he can eventually get the secret. 

So now he does the same thing with Charlie, Charlie does the same thing with Dave. Dave has the secret, so Dave can unlock this hash, get his funds, and Charlie can then do the same with Bob, and Bob can do the same with Alice. And at this point, Alice has the secret, and Dave has been successfully paid, and her having the secret is what we call a proof of payment. 

Okay, so that's kind of the payments 101 complete, and there's two things you should take away from this. 

The first thing is, it's actually just one thing, but the first thing is, there had to be this dance between Alice and Dave happening outside the network before the payment could actually happen. And then just leading on from that is, okay, why? 

Dave had to send her this thing called a hash, but there's actually quite a few other things that Dave needs to tell Alice. And that's where Bolt 11, ooh, Bolt 11 invoices comes in. 

So what Bolt 11 invoices are is this just, Bolt 11 is just specs out how Dave should structure the information that he sends to Alice. So that any wallet can just know how to structure its information, how to read the information. 

So I'm sure you've all seen a Bolt 11 invoice. It's usually displayed as a QR code or some LN something something. 

Notice this is a reg test invoice because it's not safe to pay to this, and I'll get to that in a second. 

Just diving into a little bit more about what Bolt 11 invoices look like, you'll have your prefix, that LN part, the amount that Dave expects, timestamp, some array of data that I'll get back to in a second, and a signature over the entire thing, okay. 

Now, I've already mentioned that one of the pieces of data is this payment hash, and it's very important because it's the only way payment can be secure, okay, so that's the first thing. 

The other thing is, it needs to contain a node ID, right? Alice, if she walks into Dave's coffee shop, needs to know how to find him in the network, okay, so it needs to be a node ID. 

The other thing is it optionally needs to include something called Hop Hints, because perhaps Alice's network view looks something more like this, where Dave only has private channels, and so Alice doesn't know where he is. So he has to actually tell her which UTXOs those private channels are if she's ever gonna find him in the network. So the invoice needs to include that. Maybe it contains a description, this is for one cup of coffee, and then some feature bits. So it's just a way for Dave to say to Alice, hey, I'm aware of this payment or this feature, etc. And then some other stuff, but I'm just gonna focus on those. 

Okay, cool, so what's wrong with this? So the main thing I wanna focus on, whoops, okay, wasn't meant to do that. 

Okay, it's single use, so I think people will often say, okay, invoice is single use, and people say, yes, that's insecure. But I just wanna, not insecure, insecure. 

And I just wanna demonstrate exactly why. So let's say Alice and Bob both walk into Dave's coffee shop, okay? Dave generates an invoice cuz he wants to be paid, pause. Creates a Vault 11 invoice, displays it. And Alice goes ahead, pays this as we saw before. And Dave reveals the secret, and Alice has a proof of payment for this invoice. But it just so happens that Bob saw the invoice at the same time, thought it was his maybe, and tried to start paying it. So he goes ahead, pays it. 

Now, what happens if Eve and Charlie are the same person or in cahoots with each other? Charlie can just send the secret, he knows it now, to Eve. And get the money from Bob. Bob thinks, yay, I've got proof of payment. And Dave has not been paid, right? He's only been paid once. 

Okay, so you can see that if you are paying an invoice, it's very important to you that no one else has paid this invoice before. It's very important. 

And this leads me on to my next point, which is that this is a very weak proof of payment, right? I've just said that both Alice and Bob have this proof of payment. So really, having the secret just reveals that the payment has been complete at some point, does not prove that you have paid it, okay? Okay, and then just a few other issues is, I've mentioned that Dave needs to maybe include Hop Hints in his invoice, right? If all his channels are private, and this kind of defeats the point, it's not, they're no longer really private channels. Alice can sell that information. 

So that's not ideal. The other thing is sender and receiver privacy. So if in this example that I showed where Alice pays Dave, Dave has zero privacy, right? Alice knows exactly where this payment is going. But Alice has a lot of privacy. Dave doesn't know where this payment is coming from, unless Alice now wants a refund, in which case she has to give up where she is in the network, which is not a nice decision to make. 

And then a small thing is that the Bolt11 signature that Dave makes to just prove that he created it, is over the entire thing as like a flat structure. Which is just not great if you wanna go and show that invoice and now you have to reveal all these pieces of information if you wanna show that the signature is valid. 

Okay, some bad UX. So this is a valid Bolt11 invoice, just stuffed with like 20 Hop Hints. And you can see this is not ideal, right? 

People at the back are not gonna be able to scan this with their phones. And it just demonstrates that you're kind of very much limited. There's a limit to how much information Dave can send to Alice using this method. 

And then it's just a little bit of an awkward flaw, right? 

We're used to paying directly to a static bank account that doesn't change. Now we have to tell users, okay, if you wanna be paid, someone needs to tell you first they wanna pay you. 

You tell your node, please give me your invoice. And then you send it back to the person, and then they can tell their node to pay it. 

And it's just a little bit awkward. And then another awkward flow is the withdrawal flow. So let's say Dave is an exchange, and you wanna withdraw some money from the exchange. 

Let's say you are interfacing with your exchange on your desktop, but your node is on your phone, okay? So now what you have to do is go on your phone, generate an invoice. Now you have to somehow get it to your desktop, email it to yourself, I don't know, and then send it to Dave, the exchange. 

Who can then go pay the invoice. And this is a bit of a, just an awkward flow. 

All right, now I'm gonna dive into the various proposals. 

Okay, I'll learn URL. By the way, I'm gonna be focusing mostly on the pay scenario, not the withdrawal scenario. 

Yeah, just mentioning that. 

Okay, so Allen URL is short and sweet. It basically is a set of specs that spec out all the different communications that Alice and Dave can have with each other. So all this communication that happens outside of the network, it specs that out, right? 

And what it allows us to do, so in the payments scenario, it allows Dave, so he can have this HTTP server with a static endpoint, dave.com slash payme. 

And this HTTP server will communicate with Dave's node. And now all Alice does is she sees this endpoint, can see nothing needs to change in this. It can be static, it can be printed out, it can be tattooed on yourself. Alice hits this endpoint, it'll get a fresh invoice for her and send it back. And anyone can make the same request and be pretty sure that the invoice they're getting is gonna be unique. 

So this is really cool, as you can see, short and sweet. 

And then you can have a similar idea with the withdrawal flow. And if you wanna know more about that, ask the guys upstairs, coin corner, because their cards are using Allen URL withdrawal. 

Okay, so it allows me, for example, to throw up a code like this, show it safely to an audience, right? 

I could not do this with a bot 11 invoice, it wouldn't be safe, right? 

And yeah, so it would be safe to hit the same point. Ha, so much beer for me. 

Okay, and what's also cool is that I can change the parameters of the invoice underneath this, and this doesn't need to change, right? 

So today I decide I wanna charge 10 sats for a beer. Tomorrow I decide 20 sats, and this can stay the same, okay? And then this at the bottom is just an example of a small extension to Allen URL pay, called a Lightning address. 

But it pretty much works the same, but it's easy for me to dictate over the phone, please pay to this thing that sounds like an email address, which is nice. 

Okay, so some quick pros and cons. Okay, short and sweet, as you've seen. It works already, many wallets already integrate Allen URL. 

No changes to the Lightning network or bot 11 are needed, because it's completely on top of the protocol. 

Gives us the static invoices with smaller QR codes, and it's an easy payment flow that people, it's easy for people to understand. 

And then the big cons that people will talk about is that you need this HTTP server to receive like this. 

And with that, if you're not using Tor, you don't know domain name, TLS cert, and things like that. So yeah, that's not ideal if you're running your node on your phone, for example. 

And then the other thing is, again, if you're not using Tor, it's a bit of a privacy leak for both sender and receiver. Because you now have this server, and before I've said that Alice has a lot of anonymity because Dave doesn't know where the payment's coming from. But now she's making this get request to this HTTP server first, so she's revealing her IP address there. 

Cool. Bolt 12, also known as Offers, is a very big proposal with many bells and whistles, brought forward by Rusty Russell from Blockstream. 

And yeah, at a very high level, it's kind of very similar to Allen URL, like the flow and everything and the things it provides, except for the fact that this invoice retrieval that I've been speaking about happens within the network itself, meaning you don't need this extra HTTP server. 

So people with their mobile phones can have this. And that's what makes it cool is that, yeah, you don't need anything extra. 

So when you spin up your node, this is already baked in for you. 

Okay, so just a quick high level example. What Dave will do is he'll create this thing called an offer now. 

And it can be a static offer, and it'll just contain enough information for Alice to be able to find Dave within the network. But won't have anything necessarily that needs to change a lot. So this can be static. 

And now, what Alice can use this information to find Dave within the network and send him within the network an invoice request, right? 

Dave gets this message, his node gets the message rather, and he sends the invoice back, again, through the network. And now Alice can go ahead and pay it. 

Awesome, right? Seems really cool and simple, but there's actually a lot that goes into getting this invoice retrieval within the network to happen. So let's dive into that a bit. 

So first of all, we need to be able to send a message like this. Okay, so before we, yeah. So messaging in Lightning Network, can we use some of our existing messages, right? 

So the messages we currently have in the Lightning Network is we have gossip messages. So these are the messages you wanna broadcast, you wanna kinda flood them to the whole network, right? You wanna tell the whole network, hey, new channel, new node, node channel updates, etc. 

So this is not ideal, we don't want the whole network to know that we're requesting an invoice, and we definitely don't want the whole network to get that invoice. And we quite heavily rate limit this, right? 

So a lot of nodes will batch the gossip that they send out, so it can be quite slow, right? You have to wait a bit, so this is not ideal for this scenario. 

The other types of messages that we have are the payment specific messages, right? 

So in that example I showed before, Alice has to tell Bob and Charlie how they should go about forwarding their payments on, which channels they should use. So this is a bit closer to what we want, right? Cuz it's along a specific route that Alice chooses. 

But the thing is, they're payment specific messages, right? Add HTLC, fail HTLC. And so you have to kind of create a payment to do this. 

But actually people do use this to send data across the network. What you just do is you make a fake payment, so you come up with a hash. Dave doesn't know it's coming. And, okay, Dave doesn't know it's coming. 

You attach the data you wanna send, encrypt that so that only Dave can decrypt it. Send the message to Dave, Dave gets it. He realizes he doesn't have the secret, but he can decrypt that message and just fail the payment back. 

So you can send a message like this. It's just you need to tie up liquidity, which is fine if Dave's online. But if he's offline, then you have to wait for all those CLTV expires to occur, which is not great for the network. Okay, so that means that for these offers to work, we need a whole new kind of messaging system to send these new messages. 

And that's where Onion Messaging comes in. So, about 12 builds off of Onion Messages. 

And these are basically do exactly kind of what it says. 

It allows Alice to just send a message to a specific person on the network, and she chooses the route for that message to go. 

So, this is kind of one thing that people will kind of maybe say is a con, is that the whole network kind of, to work properly, the whole network kind of needs to update to understand what Onion Messages are. 

But I would say it's not that big of an issue, because if we're gonna get a lot of benefit from this, the Lightning Network is still small, and we can still update fast, and people do update fast. So yeah. And there's many more other things you can do with Onion Messages. 

It's not just, so I've just mentioned the invoice request message and the invoice message, but you just think of, you can send any message. So there's a lot more things to come, so I'm not gonna go into that now, but watch this space. 

Okay, and then a few things you'll hear, especially on Twitter, is people will say, no, DOS, right? 

People can spam the network, people are gonna stream movies over the network, etc. So that's kind of a big argument that people will have, and just because the current proposal doesn't have a solution for that built in. 

And then the other thing is that Alice won't be able to, there's currently no way for Alice to know if her message got to Dave. So that's a little bit of an issue. 

And then the other thing is Bob and Charlie are kind of helping Alice here for free. 

Because there's no payment attached, there's no real incentive for them to pass on the message, except for the fact that, hey, I'm helping you now, so you'll help me later kind of thing. 

Which we know we do in Bitcoin anyways with transactions. 

Okay, and then I just wanna say watch the space as well, cuz in the last two weeks, there have been some proposals coming out for how we can prevent this DOS vector. 

Okay, and then a few other really cool bells and whistles that Office has is, first of all, I've spoken about Hop Hints and why that's an issue and how we're revealing the UTXOs we actually intended to keep private, so Office builds on using blinded paths instead of these Hop Hints. 

And this is basically a way for Dave to tell Alice how she can find him in the network, or rather how she can construct a route to him in the network without revealing to her where he is in the network. 

Yeah, so I'm not gonna go into detail of that, but definitely look into it. 

And this is really cool because now Alice doesn't need to know exactly where Dave is, but she can still be paid. 

And then if Alice wants a refund, she can provide a blinded path, so she doesn't have to give up her anonymity either. 

Okay, and then the other really cool thing is payer proofs. 

So what's included is that Alice will, in this in which request she sends through the network to Dave, she will include a key that belongs to herself, so a payer key. 

And now when Dave sends the invoice back, he needs to commit to the message she sent, meaning he's committing to her key. 

Meaning now when Alice gets that S value back, that does prove that she was the one who made the payment, because the invoice itself shows that she's the one who requested that invoice. 

Okay, so it's a great proof of payment. 

And then another really cool thing is just that the signature for an offer message, invoice and invoice request messages are over a Merkle root of all the fields instead of over the whole thing. 

So if you need to go reveal it to someone, you can reveal only the information you would need to reveal. 

Okay, it doesn't solve stuck-less payments, but let me just describe this first. 

So basically what a stuck payment is, is if I go to the coffee shop and I get an invoice, make a payment, it gets stuck. So I get a new invoice, make a payment through a different route, and then that succeeds. 

And then the first one also happens to succeed. There's really no way for the coffee owner to tell that I only meant to pay once. 

So Office kind of takes that into account. And what you can do is when you go and make that second invoice request, you can say, hey, I'm intending to replace this first one.

That is still maybe will succeed. So just give some plausible deniability there. 

And then what's cool is you can specify amounts in currencies other than Bitcoin, which is nice for the static case, especially when the price is fluctuating so much. So you can have a thing that's static and you're saying, okay, I'm gonna specify the currency in USD for now, just so that. 

And then it, sorry, breath. 

Just that you can keep that static and then, yeah, your wallet will need to get an exchange rate to pay it. 

And then it used to contain recurrence too. So you could say this is an offer and it means you're gonna have a monthly subscription to me, etc. But that's been removed for the time being. 

Okay, quick pros and cons. 

So it's really cool that it's lightning native, right? So you can spin up this node and already get the benefits without needing to worry about this extra server. 

Improved privacy for both sender and receiver because of blinded paths. 

And again, because of the static invoicing, we have a better UX, fantastic proof of payments. 

And then, okay, some cons is this network-wide upgrade, which I've mentioned isn't, I don't think, that big of an issue. 

And then currently the issue is this dust protection isn't, there's no set solution for it yet. 

And then another thing what people will just say is, it depends on a lot of things, right? 

I've already mentioned onion messaging, route blinding. Yeah, it's just a lot to implement if you wanna use offers.

And then it does bring some application level stuff into the network, which some people aren't happy with. Like I've said, with specifying the amounts in other currencies. 

Okay, so AMP, the last one. Atomic multi-path payments. 

So first, before I go into this, just wanna mention that the kind of true beauty of AMP is it allows you to atomically use all your channels or many of your channels to make a single payment.

And kind of as a side effect, you can use a static invoice.

So I'm gonna describe the static invoice thing first, and I'm just gonna use a single path to describe it. 

So basically, Dave creates a Bolt 11 invoice, right? 

And I've described before what kind of things the Bolt11 invoice includes. And the reason it needs to change every time is because you need this payment hash to be different every time.

But with AMP, this payment hash is meaningless, and Dave will use his feature bits to tell Alice, okay, this is an AMP invoice, you can safely pay it many times.

And now, AMP needs to obviously understand this feature. And before I go into the next point, so we have up until now assumed that Dave always needs to be the one who creates the secret and gets the hash and sends it to Alice. 

But AMP flips that on its head like he sent, and Alice will be the one who comes up with the secret, gets the hash. 

She will encrypt the secret such that only Dave can decrypt it, right? Do the whole onion encryption thing. And now she'll go ahead with a payment to this hash. 

She will attach this as metadata to that message. And when it gets to Dave, he can decrypt it and get the secret that he needs to claim the payment back, okay? 

So awesome, two things to take away from this. The first is Alice could do this without informing Dave of this beforehand.

So it allows for these spontaneous payments. And the second thing is Alice has no proof of payment here, right? So her knowing the secret means nothing because she's the one who produced the secret. 

Okay, so now I wanna just demonstrate the true kind of beauty of AMP and just show how it works visually. 

So basically Alice maybe wants to use all her channels to pay Dave, right? So what she's gonna do is she's gonna come up with this thing called a root seed, which is just a big blob of data.

And she's gonna deterministically produce four different secrets, okay? And get the hash of those secrets. Now, four because she wants to use all four of her channels, right?

And now what she's gonna do is she's gonna split that root seed into four parts. She's gonna take the first hash, make the whole payment to that hash, attach the first part of the root seed. Gets to Dave, note that he cannot claim this payment back yet, because there's no way for him to know what the secret necessary is, is for.

So only when all the parts get to Dave can he reconstruct this root seed, get the secrets again, and claim the payments.

Yeah, so that's the beauty. 

Okay, pros and cons. 

Again, static invoice. Only Alice and Dave needed to be aware of AMP. And it allows Alice to just pay, again, Dave without letting him know first.

And yeah, we can reuse both 11 invoices.

And again, you can take advantage of all your channels, which is nice, especially if it's a new user who doesn't understand channels. You can just show them one balance and they can use that entire balance.

Again, and then the big con is that there's no proof of payment. 

Okay, so just to end off, just wanna show high level comparison and just leave you with a few questions.

So just to show the lightning layers, both 1 to 11 currently kind of covers these layers, networking, messaging, payments, and invoicing. 

And then on top of that, we anyways need to do this invoice negotiation.

And on top of that, we can build up applications.

Okay, so where do these three things lie? 

So LNURL just kind of just specs out invoice negotiation, or at least LNURL pay that I've been speaking about. 

And then AMP is just a new payment type. And offers quite a big vertical stretch. It touches quite a few things.

This is not included in offers, but it depends on onion messaging. It has a different invoice. 

Ooh, sorry, gonna have to do that again. 
These are the cons of having animations. 

Okay, it has a new type of invoicing. It specs out invoice negotiation, and it does include some application level things like specifying the amount in different currency.

Okay, so a few points of discussion. Just if you're looking at arguments on Twitter, these are kind of the things that people argue about. 

Okay, so the first is how important is this proof of payment? 

So I've been speaking about how current Bolt 11 invoicing has a kind of semi-weak proof of payment. 

AMP doesn't really have one yet, right? 

Taproot and PTLCs changes things. And then offers us a really strong proof of payment.

Okay, so people, and then sorry, one side will kind of argue.

One side will kind of say, okay, it's obviously really important because when the network is super huge, you're really, really, really gonna wanna have that proof of payment.

And then the other side will kind of say, okay, but how often are people currently actually using this proof of payment? 

You know, a lot of wallets don't even show users their pre-image. So that's kind of the two sides.

Okay, the other argument is what should we and should we not include in the protocol? Yeah, where do we draw that line? 

And then the other thing is, do we need to have just one invoicing system? Like, can we not mix them somehow?

Maybe, do we want one that works for every type of user?

Like, people will say, LNURL is really good for merchants because they will anyways have a server running, but it's not really good for someone who just wants a wallet on their phone.

And then, yeah, so like, but it works really well for people who are merchants, and then offers works really well for people who have their wallet on their phone. 

So, can we have them both? Do we need just one? Yeah, so I'm not answering these questions. I'm just leaving you with them. 

Yeah, that's it. 

Cool. Any questions?

Cool. Oh, oh. Will your deck be published somewhere?
Sorry?
Will your deck be published somewhere?
I can publish them, yeah, sure, sure, will do.
Yeah, I'm glad you like them. 
Thank you. 
Cool, awesome. 
Great, thank you, Elle.

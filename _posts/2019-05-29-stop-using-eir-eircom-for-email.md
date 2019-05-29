---
tags: [security, networking]
title: "Stop using Eir/Eircom for email!"
---

Eircom is Ireland's largest telecoms provider. Formed in 1984 as Bord Telecom Éireann under the Posts and 
Telecommunications Act of 1983, Eircom has gone through many changes and controversies. Known today as Eir
and incorporated in Jersey (in France), the company is known for charging the highest line rental fees
in Europe, and has been acquired multiple times by other companies - some of which were accused of asset
stripping.

Eir provides a lot of services today - not just landline phone services and broadband, but also TV services,
home security services (known locally as Phonewatch), cellular data and mobile phone services, and more besides.
They're a big company - and this is largely down to the fact that they were once Ireland's "official" telecoms
provider and ISP, being owned and run by the government until privatisation was completed in 1999.

Eir has done well - but it hasn't been without its share of issues. The main concern of this article relates
to the privacy of your communications with Eir users (specifically, via email), as well as a small warning 
to those that do not read the fine print. 

# Encryption and Email

Email is tricky business. The technology backing email is ancient, and compatibility must be maintained as email
providers are often quite slow to adopt updated technologies. If it ain't broke, don't fix it, right?

Here's the thing: Email is old. Real old. Invented by Ray Tomlinson, email first entered limited use in the
1960s - certainly a simpler time for the internet. In the 60s, we weren't worrying about the NSA, hackers or 
bots crawling the internet and stealing our data.

In the beginning, email was unencrypted. This meant that anyone sitting between you and your intended recipient
was able to open up and read everything in your email, no questions asked. Today, people are more aware of their
privacy and that's great - we _should_ be taking steps to protect ourselves. Unfortunately, however, people are
unaware of the real privacy implications behind the simple stuff we take for granted. Email and SMS are two
inherently insecure technologies, and deserve as much attention as the services that companies like Facebook
and Twitter provide. Sadly, email is not as sexy as social media at large, and the mainstream media will never 
cover it in detail.

## The Path to Email

In today's world, we do have some additional security we can make use of when we look at email. The biggest and
most widely used of these is known as TLS - Transport Layer Security. TLS is an encryption process that happens
at various levels of the email sending process, but to understand it we need to look at how email works as a 
whole.

![How most people think of email](/assets/images/2019-05-29-stop-using-eir-eircom-for-email/cloud.png)

Most people don't think of the actual machinery of how things like email work. The above is all most people
care about - you send an email, it takes a trip across the Internet somehow, and it ends up with your recipient.
In an ideal world, this is all a user needs to know - and we're getting that point, but we're not there yet.

What if we look inside that cloud? The email we sent above takes a fairly long trip in order to reach its 
destination. At a basic level, the data you send over the internet needs to take a trip through a bunch of
places - we call these "hops". Since the cable running from your house doesn't directly connect to your friend
in Paris, we need to be able to route our data to them in the most direct way possible.

Eir's SMTP servers are at `smtp.eircom.net`. Using this address as an example, we can trace the route our data 
takes to get to that SMTP server.

```
C:\>tracert smtp.eircom.net

Tracing route to mail1.eircom.net [159.134.198.135]
over a maximum of 30 hops:

  1     1 ms    <1 ms    <1 ms  192.168.1.1
  2    13 ms    12 ms    11 ms  78.152.192.1
  3    12 ms    12 ms    12 ms  89.19.72.29
  4    13 ms    12 ms    12 ms  213.233.129.93
  5    13 ms    12 ms    12 ms  lag-40.br1.6cr.border.eircom.net [185.6.36.82]
  6    18 ms    14 ms    13 ms  86.43.12.216
  7    13 ms    14 ms    13 ms  lag-201-corea-srl-hcore1-srl.corea.srl.core.eircom.net [83.174.185.4]
  8    14 ms    13 ms    14 ms  lag-8-pe1-crz-corea-srl.pe1.crz.crz-crz.eircom.net [86.43.253.71]
  9    15 ms    21 ms    24 ms  86.43.242.22
 10    14 ms    13 ms    14 ms  mail1.eircom.net [159.134.198.135]
```

The first hop is the router we connect our phone or computer to - that's the device that provides internet to you.
After that, we're looking at the internet's routing infrastructure, until hop 5 - at this point we enter Eir's
network. After that, we make 5 more hops until we finally reach our destination at hop 10.

This is a pretty short round-trip, and you'd expect this - I am in Ireland, which is where Eir operates.
The further away the server is from you, physically, the more hops you'll tend to need to go through to reach
it.

The above is how all Internet traffic is routed. However, email adds another layer to this - SMTP relaying.

SMTP is the Simple Mail Transfer Protocol, and it's used whenever anyone sends an email. SMTP servers are 
responsible for ensuring that your email makes it to the correct place - and this is a very important job.
There are issues with this approach, but we'll just focus on those that are relevant to our privacy issue.

Assuming we're using Eir's email services and our friend uses Gmail, a simplified trip might look like this...

![A more accurate overview](/assets/images/2019-05-29-stop-using-eir-eircom-for-email/overview.png)

We've sent an email using Eir's SMTP server, and it was received by Gmail's SMTP server. Your friend then
connects to Gmail's IMAP server using the Internet Message Access Protocol, and downloads the email you
sent them.

As you can see, there are a lot of actors in this process - we are sending our email through the 10 hops we
mentioned earlier to reach Eir's SMTP server, and then our email has to make a number of extra unspecified
hops to reach Gmail's SMTP server. After that, it is retrieved by Gmail's IMAP server and it makes a final set
of hops to reach your friend's device, which has requested any potential new emails that the server might know
about.

This system works well for the most part, and it's about as efficient as we can manage right now - but there are
some security considerations to think about. The main issue is that of encryption - encryption ensures that
only our targeted recipient can read our email. In order for things to be entirely secure, we need encryption
to exist at all stages of this process:
 
* Between your phone and Eir's SMTP server
* Between Eir's SMTP server and Gmail's SMTP server
* While the email is in storage
* Between Gmail's IMAP server and your friend's phone

As mentioned before, encryption is usually provided using TLS, or Transport Layer Security. In 1999, 
[an RFC was published relating to TLS and IMAP, POP3 and ACAP](https://tools.ietf.org/html/rfc2595). Later, in
2002, [an RFC was published relating to TLS and SMTP](https://tools.ietf.org/html/rfc3207). RFCs are 
international standards that exist to provide a baseline to be implemented by software all over the world. If
an RFC exists for the technology you're using, you are expected to implement it, and there are many good reasons
for this.

This, however, is where we hit a problem. If you're a Gmail user, you may have seen a
[red lock icon](https://support.google.com/mail/answer/7039474) when reading a sent or received email. This
lock icons means that Gmail was either unable to send the email to the target server using TLS, or that the
target server sent the email to Gmail without using TLS. This means that **the email was sent or received
insecurely**, and may have been inspected or read in full during its journey across the internet.

As you may be suspecting, Eir is an email provider that brings up a lot of red lock icons for Gmail users.
There are a few possible reasons that this might happen:

* One of the servers straight up doesn't support the TLS extension
* The servers both support the TLS extension, but one of them only supports insecure versions of TLS
* The servers both support the TLS extension, but TLS negotiation is impossible due to some unknown issue

At the end of the day, what this means is that your email was transmitted without encryption at some point during
its journey across the internet. During that part of the trip, any server or device that it has to take a hop
through will be able to read the full content of the email. That's pretty bad!

I did a quick check of Eir's services, and I noted the following:

* Eir's IMAP server does support TLS
* Eir's public SMTP server does support TLS

So, this implies that Eir's backend mail delivery servers are causing a problem. Indeed, inspecting the headers
of an example email from an Eir user shows that the email was ultimately relayed to Gmail by a server known as
`mta00.svc.cra.dublin.eircom.net` - One of Eir's Mail Transfer Agents (MTAs). From this, we can surmise
that this MTA does not support sufficient levels of encryption.

It's impossible to say how many hops it took our email to reach Google's servers from there. However, Google
is quite a high value target - it isn't impossible that it may have been intercepted on its way there.

# Mitigation

So, what can we do about this? We have two realistic options:

* Add our own encryption layer - if we do this, then sender and recipient information is still exposed, but the
  content of the email remains private
* Stop using Eir's email service

The former can be achieved using GPG, the Gnu Privacy Guard. GPG is not user-friendly and requires your
recipient to also be using it, so it's not recommended for the average user. If you'd still like to investigate
this route, then I highly recommend checking out [Protonmail](https://protonmail.com/). They provide a free,
reliable email service that has GPG built-in for when you need it.

For everyone else, [Gmail](https://gmail.com) is another good option. It's free, reliable and user-friendly,
but it doesn't provide any of the GPG support that Protonmail does. If you need a free service and you use an
email client like Microsoft Office Outlook or Thunderbird, then Gmail will be the better option for you.

# Improving Email

The state of email is pretty good right now.  Google's 
[Safer Email Transparency Report](https://transparencyreport.google.com/safer-email/overview) shows that 92%
of emails both sent and received through Gmail are encrypted with TLS. That's not bad!

The true solution to this is - at least right now - to encourage all email providers to implement TLS into their
services. If you use Eir and you can't switch, then raise a complaint with their support team. If they tell you
that this isn't a real issue, don't back down - chances are that, in reading this article, you now know more than
their support agents do about this particular issue.

There have been [some attempts at replacing SMTP](https://en.wikipedia.org/wiki/Dark_Mail_Alliance), but it
seems that SMTP is here to stay - so we should all work towards making it safer for everyone.

# The Fine Print

It pays to read the fine print. It's a pain, and often these terms of service are written in language that only
a lawyer could love - but it contains information that is incredibly useful. Here's a story that shows what
may happen when you don't read it...

Back when broadband was new to rural Ireland, we signed up for dialup with Eir - or Eircom, as it was known back
then. Later on, we switched to DSL with them, a marked upgrade at the time.

I had my email with Microsoft Hotmail - which is now known as Outlook Email - and later Gmail. My mom, however,
decided to use the Eircom email address that came with our internet services. She built up a business around
this email address, and used it for all kinds of things.

One day, like many other Irish people, we realised that Eircom were shafting us. Poor service for more cost
drove us to another provider - Perlico, which is now Vodafone. We made the switch, and our service was much 
better.

As you would expect, however, mom's email ceased to be accessible. As it was provided as part of our internet
service with Eircom, and we had switched to another provider, they simply disabled access to it. This is
common sense, really - but it's something that many people don't consider, and many people have been caught
out by this.

We contacted Eircom, and after a couple weeks of complaining, regained access to the email account. It may
also be worth noting that the account was never deleted - all of the emails that were in the account were still
there, and it had received emails sent during the time we were unable to access it. I hope that this policy has
changed, but somehow, I feel like it's unlikely.

# Takeaways and Conclusions

Congratulations for making it this far! This article was not intended to be a technical overview of how TLS,
email and related technologies work - for the technical-minded among us, well, you know how to dig up that
information yourselves.

In conclusion: Security and privacy are very important. You should take steps to protect yourself from bad
actors - and you should be able to rely on your service providers to do the same. Eir does not seem to be
on top of this, unfortunately - so I advise that you switch to [Protonmail](https://protonmail.com/) for
your email service, or [Gmail](https://gmail.com/) if you don't need GPG support and use an email client.

We all have a responsibility to each other to provide security when it comes to personal information. Even
if you're not a service provider, your choice of email provider will impact the privacy of everyone you
contact, and everyone that contacts you. It's a reasonable assumption to assume that a big service
provider knows what they're doing, but that isn't always the case - vigilance is always good!

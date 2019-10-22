---
layout: post
title: "Payment bypass via parameter tampering"
date: 2019-07-24
tags: Gigs
---


I was recently on an engagement testing a client's checkout payment system. It was the type of engagement where everything seemed to be locked down and I had no findings for 2 days straight

![Give me something...](/assets/img/blog/bangkeyboard.gif)

The checkout page was a two (2) step process. The first page was a page showing the payment amount owing including options to add delivery, vouchers and coupons. The second page was the payment page where the user had to place in his credit card details or other payment methods such as paypal.

After throwing everything I could at it, I noticed something interesting about the payment method in the second step's POST request body.

{% highlight python %}
... accept_terms=1&paymentmethod=creditcard&PANnumber=xxxxxxx&CCexpiry=xxxxx....
{% endhighlight python %}

I thought "What if there were other payment methods that I could use, that could perhaps allow me to get things for free?" . 

Playing around a bit more revealed that there was indeed different methods. I noticed that if a voucher was used, and it had enough funds to complete the transaction, the payment method used in step 2 was set to "free"

{% highlight python %}
... accept_terms=1&paymentmethod=free ...
{% endhighlight python %}

I quickly repeated the whole process with a new item, intercepted the request in step 2, and removed all credit card details and replaced the method with "free".

Sure enough, the transaction went through and I was able to successfully make a transaction for free.

I reported this immediately to the client and an emergency patch was rolled out the next day.
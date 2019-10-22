---
layout: post
title: "Passcode Activity Bypass using Race Condition"
date: 2019-10-18
categories: "bug bounty"

---


An **Activity** is one of the Android's component in an app. It is the screen that the user sees on a mobile app. (For example, the setting's "screen", home "screen, etc). A simple app could have one while more complicated ones could have dozens.

From a security perspective, I normally check if Activities are **exported**, meaning if they can be directly called by other applications.

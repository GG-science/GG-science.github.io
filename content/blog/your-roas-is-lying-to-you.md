---
title: "Your ROAS is Lying to You"
date: 2024-11-12
draft: false
tags: ["experimentation", "measurement", "marketing"]
description: "Last-touch attribution was already wrong before iOS14 made it worse. Here's what incrementality testing actually measures — and how to run one."
---

In 2021, Apple gave users a button that let them opt out of cross-app tracking. Most did.

Ad platforms lost signal overnight. Facebook's reported ROAS dropped. Panicked brands cut spend. Some of them cut spend that was actually working — they just couldn't tell anymore.

But here's the thing: **last-touch attribution was already lying to you before iOS14.** Apple just made it more obvious.

---

## What last-touch attribution actually measures

Last-touch attribution gives 100% of the credit for a conversion to the last ad a customer clicked before buying.

That sounds reasonable until you think about who those customers are.

If you're running a retargeting campaign, you're serving ads to people who already visited your site, added to cart, or searched your brand name. These are your highest-intent customers. They were probably going to buy anyway. Your ad showed up last in the journey, got the credit, and your ROAS looked great.

You weren't driving purchases. You were measuring them.

This is called **selection bias** — and it makes every last-touch number suspect.

The correct question isn't "how many conversions happened after an ad click?" It's "how many *additional* conversions happened *because* of the ad?"

That gap — the difference between observed conversions and conversions that would have happened anyway — is what incrementality testing measures.

---

## What incrementality testing actually measures

An incrementality test is a controlled experiment. You split your audience into two groups:

- **Test group:** sees your ads as normal
- **Control group (holdout):** doesn't see your ads at all, or sees a neutral placeholder

You run it for 2–4 weeks. At the end, you compare conversion rates between the two groups.

The lift from test over control is your **true incremental effect** — the purchases your ads actually caused.

If your test group converts at 4.2% and your control group converts at 3.9%, your ads drove 0.3 percentage points of lift. That's real. Everything else in your ROAS number was noise.

---

## How to run a basic holdout test

You don't need a sophisticated platform to run this. Here's the minimum viable version:

**1. Define your holdout size.**
Hold out 10–20% of your target audience. Smaller holdouts give you less statistical power; larger ones cost you more potential revenue during the test.

**2. Make the groups comparable.**
Random assignment is the whole game. If your holdout is geographically skewed, or skewed toward any behavioral segment, your results are garbage. Most ad platforms (Meta, Google, TikTok) have native lift test tools that handle this for you.

**3. Choose one campaign to test.**
Don't test everything at once. Pick your highest-spend campaign — usually retargeting or broad prospecting — and test that.

**4. Run it long enough.**
Two weeks minimum. Four weeks is better. You need enough conversions in both groups to detect a real difference. If your daily conversion volume is low (under 50/day), you'll need longer.

**5. Do the math honestly.**
Incremental ROAS = (revenue from test group − revenue from control group) / ad spend on test group.

If that number is lower than your reported ROAS — and it almost always is — the gap is what you've been paying for the illusion of credit.

---

## What this usually reveals

In most holdout tests I've seen, incrementality is 40–70% of reported ROAS. Sometimes lower.

That doesn't mean the channel isn't working. It means the reported number isn't what's working.

The insight is usually sharper than the aggregate: retargeting is often the worst offender (high reported ROAS, low incrementality), while broad prospecting looks worse on paper but drives more actual new customers.

That's the kind of finding that changes where you put the next dollar.

---

## What to do next

If you've never run an incrementality test, start with your retargeting campaign. The gap there is almost always the largest — and the most actionable.

If you want help designing the test, interpreting the results, or building a measurement system that tracks incrementality on an ongoing basis, that's what I do.

[Get in touch →](mailto:gagangrewal49@gmail.com)

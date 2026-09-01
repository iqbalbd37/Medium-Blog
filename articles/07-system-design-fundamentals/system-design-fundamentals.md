# সিস্টেম ডিজাইন: যেভাবে আমি ভুল থেকে শিখেছি

### *System Design: How I Learned from My Mistakes*

[![Cover Image](images/cover.png)](images/cover.png)

---

## বাংলা সংস্করণ

### আমার গল্প শুরু হয়েছিল একটি ভুল দিয়ে

২০২০ সালে আমি আমার প্রথম স্টার্টআপ শুরু করেছিলাম - একটি ফটো শেয়ারিং অ্যাপ। সেই সময় আমার ধারণা ছিল: "একটি সার্ভার নিলেই হবে, সবকিছু সেখানে চলবে।"

আমি ঠিক তেমনই করেছি। একটি সার্ভার। সবকিছু সেখানে। ওয়েবসাইট, ডাটাবেস, আপলোড করা ছবি - সব।

প্রথম মাসে সব ঠিকঠাক চলেছিল। তারপর হঠাৎ একদিন আমার অ্যাপ Reddit-এ ভাইরাল হয়ে গেল। ১০,০০০ জন মানুষ একসাথে এলো।

**এবং সবকিছু ধ্বংস হয়ে গেল।**

সার্ভার ক্র্যাশ। ডাটাবেস ডাউন। সব ছবি হারিয়ে গেছে। আমি তখন বুঝতে পারিনি কী হচ্ছে। শুধু জানতাম - আমার অ্যাপ মারা গেছে।

সেই দিন থেকে আমি সিস্টেম ডিজাইন শেখার শুরু করেছি। এবং এই আর্টিকেলে আমি সেই শেখার অভিজ্ঞতা ভাগ করে নিচ্ছি।

---

### সিস্টেম ডিজাইন কী? সহজ ভাষায়

অনেকে সিস্টেম ডিজাইনকে জটিল করে তোলে। কিন্তু সত্যি বলি, এটি খুবই সহজ বিষয়।

**সিস্টেম ডিজাইন হলো - সঠিক জিনিসগুলো সঠিক জায়গায় রাখার কলা।**

যেমন: আপনি যখন একটি ঘর সাজান, তখন আপনি বেডরুমে বিছানা রাখেন, রান্নাঘরে চুলাই রাখেন। ভুল জায়গায় রাখলে সমস্যা হয়।

সিস্টেম ডিজাইনও তেমনই। প্রতিটি কম্পোনেন্টের একটি জায়গা আছে। সেটি সঠিক জায়গায় না রাখলে - সিস্টেম কাজ করে না।

---

### জুনিয়র, মিড-সিনিয়র, সিনিয়র - তফাৎ কোথায়?

আমি যখন জুনিয়র ছিলাম, তখন আমাকে বলা হতো: "এই ফাংশনটি লেখো।" আমি লিখতাম। শেষ।

মিড-সিনিয়র হলে বলা হয়: "ইউজাররা ফটো আপলোড করতে পারছে না, সমাধান করো।" তখন আমাকে বুঝতে হতো সমস্যাটা কী, কেন হচ্ছে, কীভাবে সমাধান করবো।

সিনিয়র হলে? "আমাদের অ্যাপ লক্ষ লক্ষ ইউজার সামলাতে পারছে না। পুরো সিস্টেমটা রিডিজাইন করো।"

**পার্থক্য হলো:**
- জুনিয়র: ইনপুট-আউটপুট জানা
- মিড-সিনিয়র: সমস্যা বোঝা এবং সমাধান খোঁজা
- সিনিয়র: পুরো সিস্টেম ডিজাইন করা

---

### আমার ফটো অ্যাপ থেকে যা শিখেছি

আমার অ্যাপে প্রথম সমস্যা হয়েছিল যখন হঠাৎ ট্রাফিক বেড়ে গেল। সার্ভার CPU ১০০% হয়ে গেল। পেজ লোড হতে ১০ সেকেন্ড লাগছে।

**প্রথম শেখা:** যখন একটি সার্ভার সামলাতে পারে না, তখন দ্বিতীয়টি যোগ করো। কিন্তু প্রশ্ন এলো - দুটি সার্ভার থাকলে, ইউজারের অনুরোধ কোনটিতে যাবে?

**এই সমস্যার সমাধান হলো লোড ব্যালেন্সার।**

লোড ব্যালেন্সার হলো একটি ট্রাফিক পুলিশ। সে ঠিক করে প্রতিটি অনুরোধ কোন সার্ভারে যাবে। যদি একটি সার্ভার মারা যায়, তাহলে সে অন্য সার্ভারে পাঠায়।

---

### ডাটাবেস কেন জ্বলে উঠছিল?

তারপর আমি দেখলাম ডাটাবেস slow হয়ে যাচ্ছে। কেন? কারণ প্রতিটি ইউজার একই জিনিস খুঁজছিল - ট্রেন্ডিং ফটো, জনপ্রিয় প্রোফাইল। ডাটাবেস প্রতিবার সেই কোয়েরি নতুন করে চালাচ্ছিল।

**দ্বিতীয় শেখা:** জনপ্রিয় জিনিসগুলো আলাদা করে রাখো। একে বলে **ক্যাশিং**।

ক্যাশ হলো একটি দ্রুত মেমরি। যেমন আপনি যদি প্রতিদিন একই বই পড়েন, তাহলে সেটি ডেস্কে রাখলে আর বই ঘরে গিয়ে আনতে হয় না। ক্যাশও তেমনই - জনপ্রিয় ডাটা কাছে রাখে।

---

### সবকিছু একটি জায়গায় রাখা কেন ভুল?

আমার সার্ভারের ডিস্ক একদিন মারা গেল। সব ছবি হারিয়ে গেল। কারণ সবকিছু ছিল একটি সার্ভারে।

**তৃতীয় শেখা:** ডাটা একাধিক জায়গায় রাখো। একে বলে **রেপ্লিকেশন** এবং **ব্যাকআপ**।

এখন আমার সিস্টেমে:
- ডাটাবেসের দুটি কপি আছে (মাস্টার এবং স্লেভ)
- ছবিগুলো AWS S3-এ সেভ হয় (যেটি অটোমেটিক্যালি তিনটি জায়গায় কপি করে)

**একে বলে সিঙ্গল পয়েন্ট অফ ফেইলার এড়ানো।**

---

### ফাংশনাল বনাম নন-ফাংশনাল: দুটি ভিন্ন প্রশ্ন

ডিজাইন শুরু করার আগে দুটি ধরনের প্রশ্ন করতে হয়:

**প্রথম প্রশ্ন: অ্যাপটি কী করবে?**
- ইউজার সাইন আপ করবে
- ফটো আপলোড করবে
- ফলো করবে
- ফিড দেখবে
- লাইক-কমেন্ট করবে

**দ্বিতীয় প্রশ্ন: অ্যাপটি কতটা ভালো কাজ করবে?**
- ফিড কত দ্রুত লোড হবে? (২০০ms এর কম হোক)
- ছবি নিরাপদ থাকবে? (ডিস্ক ফেইল হলেও)
- কত ইউজার সামলাতে পারবে? (লক্ষ লক্ষ)
- খরচ কত হবে?

**মনে রাখুন:** ইউজার কখনো বলে না "আমাকে ক্যাশ দাও।" সে বলে "আমাকে দ্রুত অ্যাপ দাও।" ক্যাশ হলো সেই দ্রুততার একটি উপায়।

---

### HLD বনাম LLD: কোন লেভেলে কথা বলছেন?

একবার আমি ইন্টারভিউতে গিয়েছিলাম। ইন্টারভিউয়ার বললেন: "ইনস্টাগ্রাম ডিজাইন করো।"

আমি সাথে সাথে শুরু করলাম: "ফাস্ট API-তে একটি লাইক ফাংশন লিখব, সেটি ডাটাবেসে সেভ করব..."

ইন্টারভিউয়ার বললেন: "থামো। আমি জিজ্ঞাসা করেছি পুরো সিস্টেমটা কীভাবে ডিজাইন করবে - সার্ভার, ডাটাবেস, ক্যাশ, CDN - এগুলো কীভাবে একসাথে কাজ করবে।"

**ভুল:** আমি LLD-র প্রশ্নে HLD-র উত্তর দিচ্ছিলাম।

**HLD (হাই-লেভেল ডিজাইন):** বড় ছবি। কোন কম্পোনেন্ট আছে, সেগুলো কীভাবে সংযুক্ত।

**LLD (লো-লেভেল ডিজাইন):** ছোট ফিচার। কোন ফাংশন, কোন ক্লাস, কোন ডাটা স্ট্রাকচার ব্যবহার হবে।

**ইন্টারভিউ টিপস:** প্রথমে জিজ্ঞাসা করুন - "HLD চান নাকি LLD?" একটি প্রশ্নে ইন্টারভিউয়ারকে খুশি করে দিতে পারেন।

---

### মনোলিথ বনাম মাইক্রোসার্ভিস: বাস্তবতা কী?

অনেকে বলেন: "মনোলিথ খারাপ, মাইক্রোসার্ভিস ভালো।"

**সত্য:** এটি সম্পূর্ণ ভুল।

আমার বন্ধুর একটি স্টার্টআপ আছে। ৫ জন ইঞ্জিনিয়ার। তারা ৩০টি মাইক্রোসার্ভিস চালাচ্ছে। ফলাফল? তারা প্রোডাক্ট ডেভেলপ করার চেয়ে বেশি সময় নেটওয়ার্ক ডিবাগিংয়ে কাটাচ্ছে।

**বাস্তবতা:** বেশিরভাগ কোম্পানি চালায়:
- একটি মনোলিথ (পুরো অ্যাপ)
- প্লাস কয়েকটি আলাদা সার্ভিস

মনোলিথ ভালো কারণ:
- সবকিছু এক জায়গায়
- ডিবাগিং সহজ
- ডেপ্লয়মেন্ট সহজ

মাইক্রোসার্ভিস ভালো কারণ:
- আলাদাভাবে স্কেল করা যায়
- আলাদা দল আলাদাভাবে কাজ করতে পারে

**সিদ্ধান্ত:** ছোট হলে মনোলিথ দিয়ে শুরু করো। প্রয়োজন হলে আলাদা করো।

---

### স্কেলিং: বড় করো নাকি বেশি করো?

**ভার্টিকাল স্কেলিং:** একটি সার্ভারকে বড় করো।
- CPU বাড়াও
- RAM বাড়াও
- সহজ, কোড পরিবর্তন লাগে না
- কিন্তু: একটি সীমা আছে। সবচেয়ে বড় সার্ভারটিও একদিন যথেষ্ট হবে না

**হরিজন্টাল স্কেলিং:** অনেকগুলো ছোট সার্ভার চালাও।
- প্রয়োজন হলে আরেকটি যোগ করো
- কোনো সীমা নেই
- একটি মারা গেলে অন্যগুলো কাজ করে

**আমার অভিজ্ঞতা:** আমি প্রথমে ভার্টিকাল স্কেলিং দিয়ে শুরু করেছি। $১০/মাস সার্ভার থেকে $১০০/মাস সার্ভারে গিয়েছি। তারপর বুঝেছি এটি যথেষ্ট নয়। তারপর ৫টি ছোট সার্ভারে যাই।

**নিয়ম:** প্রথমে সহজ পথ ধরো। প্রয়োজন হলে জটিল পথে যাও।

---

### স্টেটলেস বনাম স্টেটফুল: যেখানে আমি ভুল করেছি

আমার অ্যাপে একটি বাগ ছিল - ইউজাররা হঠাৎ লগ আউট হয়ে যাচ্ছিল। কেন?

**কারণ:** আমি লগইন সেশন সার্ভারের মেমরিতে সেভ করছিলাম। ইউজার যখন লগইন করল, সেশন সার্ভার ১-এ গেল। পরের রিকোয়েস্ট গেল সার্ভার ৫-তে। সার্ভার ৫ জানে না ইউজার কে। ফলাফল: লগ আউট।

**সমাধান:** সেশনগুলো একটি **শেয়ার্ড স্টোর**-এ রাখো (যেমন Redis)।

**স্টেটলেস সার্ভার মানে:** সার্ভার কিছুই মনে রাখে না। প্রতিটি রিকোয়েস্ট নিজে সব জানিয়ে আসে। সার্ভার কাজ করে, উত্তর দেয়, ভুলে যায়।

**এটিই স্কেলিং-এর প্রথম নিয়ম।**

---

### DNS, HTTP, HTTPS - যখন ইউজার "Enter" চাপে

ইউজার যখন আপনার ওয়েবসাইটের URL টাইপ করে Enter চাপে, তখন কী হয়?

১. **DNS Lookup:** ব্রাউজার জিজ্ঞাসা করে - "এই URL-এর IP address কত?"
২. **HTTPS Request:** ব্রাউজার সার্ভারে একটি চিঠি পাঠায় - "আমাকে হোমপেজ দাও।"
৩. **Response:** সার্ভার উত্তর দেয় - "এই রে হোমপেজ।"

**HTTPS কেন দরকার?** HTTP-তে ডাটা পাঠালে মাঝখানে কেউ পড়তে পারে। HTTPS-তে সব encrypted থাকে। যেমন: Wi-Fi হ্যাকার যদি আপনার পাসওয়ার্ড চুরি করতে চায়, HTTPS তা রোধ করে।

**Latency বনাম Bandwidth:**
- **Latency:** একটি রিকোয়েস্ট যাওয়া এবং উত্তর আসতে কত সময় লাগছে
- **Bandwidth:** একসাথে কত ডাটা পাঠানো যায়

**উদাহরণ:** ফিডে ৩০টি ছোট ছবি আছে। ছবিগুলো ছোট, তাই bandwidth সমস্যা নয়। কিন্তু প্রতিটি ছবি আলাদাভাবে লোড হলে ৩০বার round trip হবে - latency সমস্যা!

**সমাধান:** CDN (Content Delivery Network)। ছবিগুলো বিশ্বের বিভিন্ন জায়গায় রাখো। ইউজার যেখানে আছে, সেখান থেকে দাও।

---

### লোড ব্যালেন্সার: ট্রাফিক পুলিশ

১০টি সার্ভার আছে। ইউজার এলো। কোনটিতে পাঠাবে?

**লোড ব্যালেন্সার** সেই পুলিশ। সে ঠিক করে:

**রাউন্ড রবিন:** ১, ২, ৩, ১, ২, ৩... এভাবে ঘুরে ঘুরে পাঠায়।

**লিস্ট কানেকশনস:** যে সার্ভারে কম কাজ চলছে, সেখানে পাঠায়।

**হেলথ চেক:** প্রতি কয়েক সেকেন্ডে জিজ্ঞাসা করে - "তুমি কি বেঁচে আছ?" মারা গেলে সেই সার্ভারে আর পাঠায় না।

**ইন্টারভিউ টিপস:** লোড ব্যালেন্সার নিজেও মারা যেতে পারে। তাই দুটি রাখো - একটি মারা গেলে অন্যটি কাজ করবে।

---

### মনোলিথ থেকে মাইক্রোসার্ভিসে যাওয়া: কখন?

আমার অ্যাপ বড় হতে থাকল। একটি সমস্যা দেখা দিল - ফিড লোডিং ধীর হচ্ছে, কিন্তু ফটো আপলোড দ্রুত হচ্ছে। দুটো আলাদা সমস্যা।

আমি ফিড লোডিংকে আলাদা সার্ভিস করে দিলাম। এখন ফিড সার্ভিস ১০টি সার্ভারে চলছে, আপলোড সার্ভিস ৩টি সার্ভারে। আলাদাভাবে স্কেল করা যাচ্ছে।

**কখন আলাদা করবে:**
- যখন একটি ফিচারের স্কেলিং প্রয়োজন অন্যটি থেকে আলাদা
- যখন দল বড় হয়ে একটি কোডবেসে কাজ করা কঠিন হয়ে পড়ে

**কখন আলাদা করবে না:**
- যখন দল ছোট (৫-১০ জন)
- যখন স্কেলিং সমস্যা নেই

---

### শেষ কথা: যা আমি শিখেছি

সিস্টেম ডিজাইন হলো শুধু ডায়াগ্রাম আঁকা নয়। এটি হলো:

- সমস্যা চেনা
- সঠিক প্রশ্ন করা
- সঠিক কম্পোনেন্ট বাছাই
- প্রতিটি সিদ্ধান্তের খরচ বোঝা
- ভুল থেকে শেখা

**ইন্টারভিউতে মনে রাখুন:**
- প্রথম ৫ মিনিটে প্রশ্ন করুন
- প্রয়োজনীয়তা নিশ্চিত করুন
- প্রতিটি কম্পোনেন্ট যুক্তিসহ ব্যাখ্যা করুন

**সিস্টেম ডিজাইন শেখা যায় না ডায়াগ্রাম মুখস্থ করে; শেখা যায় বিভিন্ন কম্পোনেন্ট ব্যর্থ হতে দেখে।**

আমার সবচেয়ে বড় শেখা হলো: **ভুল করো, কিন্তু একই ভুল দুবার কোরো না।**

---

## English Version

### My Story Started with a Mistake

In 2020, I started my first startup - a photo sharing app. My thinking was simple: "One server will do, everything will run there."

I did exactly that. One server. Everything on it. Website, database, uploaded photos - all of it.

The first month went smoothly. Then one day, my app went viral on Reddit. 10,000 people showed up at once.

**And everything crashed.**

Server down. Database offline. All photos lost. I didn't understand what was happening. I only knew - my app was dead.

That day, I started learning system design. And in this article, I'm sharing what I learned.

---

### What is System Design? In Simple Terms

Many people make system design sound complicated. But honestly, it's quite simple.

**System design is the art of putting the right things in the right place.**

Think of it like arranging a room. You put the bed in the bedroom, the stove in the kitchen. Put them in the wrong place, and things don't work.

System design is the same. Every component has its place. Put it in the wrong place - and the system fails.

---

### Junior, Mid-Senior, Senior - What's the Difference?

When I was a junior, I was told: "Write this function." I wrote it. Done.

As a mid-senior: "Users can't upload photos, fix it." Then I had to understand what the problem was, why it was happening, and how to solve it.

As a senior? "Our app can't handle millions of users. Redesign the entire system."

**The difference:**
- Junior: Knows input and output
- Mid-Senior: Understands the problem and finds solutions
- Senior: Designs the entire system

---

### What I Learned from My Photo App

My first problem came when traffic spiked. Server CPU hit 100%. Pages took 10 seconds to load.

**First lesson:** When one server can't handle it, add a second. But then - with two servers, which one handles the user's request?

**The answer is a load balancer.**

A load balancer is a traffic police. It decides which server handles each request. If one server dies, it sends traffic elsewhere.

---

### Why Was the Database Catching Fire?

Then I noticed the database was slowing down. Why? Because every user was looking for the same things - trending photos, popular profiles. The database ran the same query from scratch every time.

**Second lesson:** Keep popular things separate. This is called **caching**.

A cache is fast memory. Like keeping a book on your desk instead of walking to the bookshelf every time. The cache keeps popular data close.

---

### Why is Keeping Everything in One Place Wrong?

One day, my server's disk died. All photos were lost. Because everything was on one server.

**Third lesson:** Keep data in multiple places. This is called **replication** and **backup**.

Now my system has:
- Two copies of the database (master and slave)
- Photos stored on AWS S3 (which automatically copies to three locations)

**This is called avoiding a Single Point of Failure.**

---

### Functional vs Non-Functional: Two Different Questions

Before designing, you need to ask two types of questions:

**First question: What will the app do?**
- Users can sign up
- Users can upload photos
- Users can follow others
- Users can see the feed
- Users can like and comment

**Second question: How well will the app do it?**
- How fast does the feed load? (Under 200ms)
- Are photos safe? (Even if disk fails)
- How many users can it handle? (Millions)
- What will it cost?

**Remember:** Users never say "Give me a cache." They say "Give me a fast app." Caching is one way to achieve that speed.

---

### HLD vs LLD: Which Level Are You Talking At?

Once I went to an interview. The interviewer said: "Design Instagram."

I immediately started: "I'll write a like function in FastAPI, save it to the database..."

The interviewer said: "Stop. I asked how you'd design the whole system - servers, databases, caches, CDNs - how they work together."

**My mistake:** I was answering an LLD question with an HLD response.

**HLD (High-Level Design):** The big picture. Which components exist, how they connect.

**LLD (Low-Level Design):** Small features. Which function, which class, which data structure.

**Interview tip:** Always ask first - "Do you want HLD or LLD?" One question can make the interviewer happy.

---

### Monolith vs Microservices: What's the Reality?

Many say: "Monolith is bad, microservices are good."

**Truth:** That's completely wrong.

My friend has a startup. 5 engineers. They're running 30 microservices. Result? They spend more time debugging networks than building product.

**Reality:** Most companies run:
- One monolith (the whole app)
- Plus a few separate services

Monolith is good because:
- Everything in one place
- Easy to debug
- Easy to deploy

Microservices are good because:
- Can scale independently
- Teams can work independently

**Decision:** Start with monolith if small. Split when needed.

---

### Scaling: Make Bigger or Make More?

**Vertical Scaling:** Make one server bigger.
- More CPU
- More RAM
- Simple, no code changes needed
- But: there's a limit. Even the biggest server won't be enough one day

**Horizontal Scaling:** Run many small servers.
- Add another when needed
- No upper limit
- If one dies, others keep working

**My experience:** I started with vertical scaling. Went from $10/month to $100/month server. Then realized it wasn't enough. Then moved to 5 small servers.

**Rule:** Start simple. Move to complex when needed.

---

### Stateless vs Stateful: Where I Made a Mistake

My app had a bug - users were suddenly getting logged out. Why?

**Cause:** I was saving login sessions in server memory. When a user logged in, the session went to server 1. The next request went to server 5. Server 5 doesn't know who the user is. Result: logged out.

**Solution:** Store sessions in a **shared store** (like Redis).

**Stateless server means:** The server remembers nothing. Each request brings everything it needs. The server does its work, sends the answer, and forgets.

**This is the first rule of scaling.**

---

### DNS, HTTP, HTTPS - When the User Hits "Enter"

When a user types your website URL and hits Enter, what happens?

1. **DNS Lookup:** Browser asks - "What's the IP address for this URL?"
2. **HTTPS Request:** Browser sends a letter to the server - "Give me the homepage."
3. **Response:** Server replies - "Here's your homepage."

**Why HTTPS?** With HTTP, data travels as readable text. Anyone on the path can read it. With HTTPS, everything is encrypted. If a Wi-Fi hacker tries to steal your password, HTTPS prevents it.

**Latency vs Bandwidth:**
- **Latency:** How long a request takes to go and come back
- **Bandwidth:** How much data can be sent at once

**Example:** Feed has 30 small photos. Photos are small, so bandwidth isn't the issue. But if each photo loads separately, that's 30 round trips - latency problem!

**Solution:** CDN (Content Delivery Network). Store photos around the world. Give users what's closest to them.

---

### Load Balancer: The Traffic Police

You have 10 servers. A user arrives. Which one gets the request?

A **load balancer** is that police. It decides:

**Round Robin:** 1, 2, 3, 1, 2, 3... rotating through.

**Least Connections:** Send to the server with the fewest active requests.

**Health Check:** Every few seconds, asks each server - "Are you still alive?" If one dies, traffic stops going there.

**Interview tip:** The load balancer itself can die too. So run two - if one fails, the other takes over.

---

### When to Move from Monolith to Microservices?

My app kept growing. A problem appeared - feed loading was slow, but photo uploads were fast. Two different problems.

I made the feed loading a separate service. Now the feed service runs on 10 servers, the upload service on 3. They scale independently.

**When to split:**
- When one feature needs different scaling than another
- When the team grows and one codebase becomes hard to work with

**When NOT to split:**
- When the team is small (5-10 people)
- When there's no scaling problem

---

### Final Thoughts: What I Learned

System design isn't about memorizing diagrams. It's about:

- Understanding the problem
- Asking the right questions
- Choosing the right components
- Understanding the cost of each decision
- Learning from mistakes

**Interview Tips:**
- Spend the first 5 minutes asking questions
- Clarify requirements
- Justify each component with reasoning

**You don't learn system design by memorizing diagrams. You learn it by watching different components fail.**

My biggest lesson: **Make mistakes, but never make the same mistake twice.**

---

**Tags:** #SystemDesign #SoftwareEngineering #Architecture #Scalability #LoadBalancing #Caching #Microservices #TechInterview #BackendDevelopment #CloudComputing

# Telegram 📬
*The App That Accidentally Became the Internet's Town Square, Underground Market, and Last Line of Defence (Depending on Who You Ask)*

## The Origin Story: Born from Paranoia (The Good Kind)

It's 2013. Two brothers are in Russia. The FSB — Russia's intelligence agency and the people who definitely aren't reading your texts — sends men to Pavel Durov's door demanding he hand over data on political activists. It's the second time this has happened.

Pavel Durov decides he needs a messaging app that no government can wiretap.

And so Telegram is born. Not from a garage or a dorm room, but from the very specific motivation of two Russian tech billionaires who don't want the state reading their messages. As founding stories go, this one has *stakes*.

### The Durov Brothers: A Two-Man Supergroup

**Pavel** (see his full bio in the people section) provided the vision, funding, and chaotic energy.

**Nikolai Durov** is the quieter, scarier one. A genuine mathematical prodigy who won gold at the International Mathematical Olympiad twice, Nikolai wrote most of the core cryptography and protocol architecture for Telegram essentially by himself. He's described as reclusive, brilliant beyond measure, and entirely uninterested in public attention.

If Pavel is the face, Nikolai is the brain. If Pavel is the fireworks, Nikolai is the physics.

Together, they had the rare combination of technical depth and product ambition that makes for truly world-changing software.

## The Launch: August 2013

Telegram launched on **August 14, 2013** for iOS (Android came a few months later). The app was free, had no ads, and was positioned around two pillars:

1. **Speed**: Messages should arrive instantly, always
2. **Security**: Your messages are yours, not a product to be monetised

It immediately found a user base among the privacy-conscious, the politically active, and journalists operating in difficult environments. But it also found... everyone else. Because it turned out that "fast, free, and not surveilling you" is a pretty compelling pitch regardless of your political situation.

## The Technology Stack: Built Different

Telegram didn't just build a messaging app. They built infrastructure.

### MTProto: The Protocol Nobody Asked For (But Everyone Needed)

Nikolai Durov designed **MTProto** (Mobile Transport Protocol) — Telegram's proprietary encryption protocol — from scratch. This was a controversial decision. Security researchers would have preferred they used established protocols like Signal's. Pavel's response was essentially "our mathematician brother is smarter than your mathematician brother."

MTProto uses:
- **AES-256** symmetric encryption
- **RSA 2048-bit** key exchange
- **SHA-256** for message authentication

The debate about whether MTProto is as secure as Signal Protocol has never fully resolved, mostly because cryptographers have strong opinions and the internet is infinite.

### The Architecture That Made Speed Possible

Telegram was designed around a few core architectural principles that were unusual in 2013:

**1. Distributed Data Centres**
Rather than a single data centre, Telegram built multiple servers across different jurisdictions from early on. Your data isn't just in one place. This was partly for performance (servers closer to users = faster messages) and partly for resilience against any single government demanding data.

**2. MTProto's Server-Client Architecture**
Unlike end-to-end encrypted apps where the server can't read anything, Telegram's standard cloud chats are encrypted between client and server — meaning Telegram *can* technically read them. This was a deliberate design choice to enable features like:
- Message syncing across devices
- Searchable message history
- Bots with access to message content

The truly private option — **Secret Chats** — uses genuine end-to-end encryption with forward secrecy, but these don't sync across devices. Pavel's argument is that most people want convenience AND privacy, not just one.

**3. The Client is Open Source (The Server Isn't)**
Telegram's apps are open source. The server code is not. This lets security researchers verify the app behaves correctly without revealing infrastructure details. Critics call this "security theatre." Pavel calls it "pragmatic."

### The Tech Stack Snapshot (Early Days)

- **Language**: Primarily C++ for core components (Nikolai's domain), later expanding to Java/Kotlin for Android and Swift for iOS
- **Infrastructure**: Custom-built data centres, later supplemented with cloud (though Telegram has historically been cagey about exactly what runs where)
- **Database**: Custom storage solutions rather than off-the-shelf databases — Nikolai didn't trust that existing databases could handle the throughput they needed
- **File Storage**: Their own CDN implementation for media, which is why Telegram handles large files with unusual grace

## The Features That Changed the Game

Telegram consistently shipped features that competitors took years to copy, or never did:

### Channels (2015) 📢
A one-to-many broadcast mechanism where an admin publishes to unlimited subscribers. This sounds simple but it was transformative. Suddenly:
- Journalists could publish to audiences without a platform's algorithm deciding reach
- Activists could organise without Facebook knowing
- Crypto scammers could reach millions (more on this later)
- News channels in authoritarian countries could operate with relative safety

Telegram Channels became the dominant communication medium in Iran, Belarus, Russia, Ukraine, and dozens of other countries where traditional media is compromised.

### Groups with 200,000 Members (2015→) 👥
WhatsApp capped groups at 256 people. Telegram said "hold our borscht" and eventually raised the limit to 200,000 members.

Two hundred thousand people. In one chat. With polls, reactions, pinned messages, thread replies, and admin controls. 

This turned Telegram groups into something between a social network, a forum, and a town hall. Communities that would have existed across Discord, Facebook, and Reddit instead consolidated on Telegram.

### Bots (2015) 🤖
Telegram's Bot API was years ahead of anything else. With simple HTTP commands, developers could build:
- Customer service bots
- Games
- Payment processors
- Mini-applications
- News aggregators
- Voting systems

The bot ecosystem became enormous and self-sustaining. Telegram essentially created an app store without calling it an app store, and without taking a 30% cut (until much later).

### File Sharing: 2GB Files, No Questions Asked 📁
While WhatsApp limited you to 16MB and iMessage choked on anything big, Telegram offered enormous file transfers essentially from the start, with limits eventually reaching 2GB. This made Telegram a de facto piracy infrastructure in many regions — a side effect that pleased nobody officially and everyone privately.

### Voice and Video (2017+) 🎙️
Telegram added voice calls in 2017, video calls in 2020, and voice chats (group audio) in 2020 during COVID lockdowns. The timing of group voice chats was either impeccable or planned — either way, it captured massive audience right as everyone was desperate for social audio.

## The Business Model: "We'll Figure It Out Eventually"

For most of its history, Telegram had essentially no business model. Pavel funded it personally from his VK exit money — reportedly burning through tens of millions per year to keep servers running.

The company sold **no advertising**. Charged **no subscription fees**. Collected **no user data** to sell.

This was either:
a) Principled and admirable
b) Financially unsustainable
c) Both, obviously

### The TON Blockchain Disaster (2018-2020) ⛓️

In 2018, Telegram tried to solve the money problem by raising **$1.7 billion** in a private token sale for the **Telegram Open Network (TON)** — a blockchain platform that would integrate a cryptocurrency (GRAM tokens) directly into Telegram.

The vision was ambitious: a blockchain fast enough for daily transactions, integrated with a messaging app that already had hundreds of millions of users. It could have been genuinely transformative.

Then the SEC sued them. The US Securities and Exchange Commission argued that GRAM tokens were unregistered securities. In 2020, after a lengthy legal battle, Telegram settled with the SEC for $18.5 million and agreed to return $1.2 billion to investors.

The dream died. But Telegram gave the TON project to the open-source community, which later revived it independently as "The Open Network." It's now a thriving ecosystem that Telegram has quietly re-adopted. The cryptocurrency built on it is called **Toncoin (TON)** and is now officially integrated into Telegram. Full circle. Just... more slowly, and with more lawyers. ⚖️

### Premium (2022) and Ads (2021-ish)

In 2021, Telegram quietly launched **Sponsored Messages** — text-only ads shown in large public channels (100,000+ subscribers). No targeting based on personal data. No tracking. Just contextual placement based on channel topic.

In 2022, **Telegram Premium** launched at $4.99/month, offering:
- Faster downloads
- More reactions
- Larger file uploads (4GB)
- Exclusive stickers and emoji
- No ads in channels

This was actually a sustainable business model and it worked. Telegram reportedly crossed **$300 million in annual revenue** by 2024, with Premium subscribers in the tens of millions.

## The 30 Employees That Broke Economics

This is the part that makes other tech companies want to cry into their org charts.

In 2022, Telegram announced it had approximately **30 employees** serving **700 million monthly active users**.

To put that in context:
- Twitter had ~7,500 employees when Elon Musk bought it and said he'd cut it to 1,500 (which he thought was brutal)
- WhatsApp had ~55 employees when Facebook acquired them for $19 billion (already legendary for leanness)
- Snapchat had over 5,000 employees for a comparable user base

Telegram had *thirty people*.

### How Is This Possible?

**1. No Content Moderation at Scale (The Controversial Part)**
Traditional social platforms spend enormous resources on trust and safety teams. Telegram historically had almost none. Content moderation was minimal to non-existent. This is both how they stayed lean AND why they've been implicated in facilitating serious crimes. You cannot fully separate these facts.

**2. Automation Everywhere**
The Bot API wasn't just for users — Telegram used bots extensively internally. Customer support, reports processing, infrastructure monitoring — bots doing work that would require teams at other companies.

**3. The Nikolai Effect**
When your chief architect is one of the most gifted systems programmers alive and has built the infrastructure to be self-healing and scalable from the ground up, you need fewer people to maintain it. Good initial architecture is worth more than 1,000 engineers fighting bad architecture.

**4. No Advertising Ops**
Traditional social networks need armies of people managing advertiser relationships, brand safety, targeting systems, and ad fraud. Telegram had almost none of this.

**5. Pavel's Management Philosophy**
Pavel has been extremely public about his belief that small teams move faster and that most corporate processes are theatre. He hired obsessively for skill and trusted people to own domains completely. No middle management, no product review committees, no design-by-committee.

**6. No Enterprise Sales, No B2B**
No salespeople. No account managers. No enterprise tier. Just the product.

The result was an organisation with the overheads of a small startup and the scale of a global platform.

## The Growth Story: From Privacy App to Global Town Square

### The Snowden Effect (2013)
Edward Snowden's NSA revelations dropped in June 2013 — just two months before Telegram launched. The world suddenly cared about surveillance. Telegram was perfectly positioned.

### The WhatsApp Exodus Events
Every time WhatsApp had a privacy scare or terms of service update, Telegram gained tens of millions of users within days. The biggest: WhatsApp's January 2021 privacy policy update sent 25 million users to Telegram in 72 hours.

Signal also benefited, but Telegram got more because it offered more features.

### Geopolitical Relevance
Telegram became the *de facto* communication infrastructure for:
- **Iran protests** (2019, 2022) — where it was simultaneously used by protesters AND government
- **Hong Kong protests** (2019) — organisers coordinated entire logistics on Telegram
- **Belarus protests** (2020) — the Nexta channel on Telegram became a main opposition news source
- **Ukraine-Russia war** (2022–) — both sides, plus journalists, plus citizens, plus intelligence services, all on Telegram simultaneously
- **COVID misinformation** — unfortunately also Telegram, extensively

The app became simultaneously a tool for liberation and a vector for everything wrong with decentralised information. A platform truly without an editorial point of view ends up reflecting humanity — all of it.

### The Numbers
- **2013**: Launch
- **2014**: 35 million users
- **2016**: 100 million users
- **2018**: 200 million users
- **2020**: 400 million users
- **2021**: 500 million users (post-WhatsApp exodus: +25M in three days)
- **2022**: 700 million users
- **2024**: 950 million+ users (over 1 billion by some estimates)

## The $30 Billion Valuation: How?

Telegram has never done an IPO and never published detailed financials, but analysts estimate its value at $20–30+ billion. Here's the logic:

**Revenue (2024 estimates)**: ~$300–500M annually
- Premium subscriptions
- Sponsored messages
- Fragment (their username marketplace built on TON blockchain)
- Business features

**Comparable Valuations**: Discord was valued at $15B with far fewer users. Snap went public at similar scale. Twitter was acquired for $44B.

**The Premium**: Telegram has something almost no platform has — genuine user trust that it won't sell them out to advertisers. In a surveillance capitalism world, that's increasingly rare and valuable.

**The Platform Optionality**: With 1 billion users and a functional payments/blockchain layer (TON), Telegram has more possible future revenue streams than almost any private company.

**The Pavel Discount**: On the other hand, the company is entirely dependent on one mercurial founder who has been arrested, is under investigation, and runs the thing like a personal fiefdom. Investors price in eccentricity risk accordingly.

## The Dark Side of the Cabinet 🌑

No honest breakdown of Telegram omits this part.

Telegram's privacy features and minimal moderation made it the platform of choice for:
- Drug marketplace channels
- Child sexual abuse material distribution networks (a deeply serious problem)
- Terrorism coordination and propaganda
- Scam operations at industrial scale
- Political misinformation campaigns

Pavel's position for most of Telegram's history was essentially libertarian absolutism: the platform is a neutral pipe, encryption protects human rights, and moderation at scale is surveillance capitalism by another name.

This worked until it didn't. French law enforcement's 2024 arrest of Pavel Durov was the reckoning that had been building for years. The subsequent policy changes — more cooperation with law enforcement, expanded moderation, easier reporting — represented a meaningful shift in Telegram's foundational philosophy.

Whether this is a betrayal of Telegram's principles or a necessary evolution into responsible adulthood depends heavily on what you were using Telegram for.

## The Verdict: What Is Telegram, Actually?

After more than a decade, Telegram is simultaneously:

🗞️ **A news platform** — where millions get their primary information diet

💬 **A messaging app** — still the best large-group communication tool

🤖 **A bot/automation platform** — powering countless services

🏪 **A marketplace** — for digital goods, channels, usernames

🔒 **A privacy tool** — for journalists, activists, and dissidents

🕳️ **A lawless frontier** — where bad actors operate at scale

No other app holds all these identities simultaneously. It's not social media, not a messaging app, not a news feed — it's all of them, run by a team of thirty people from a company registered in Dubai, funded by a philosophy as much as a business model.

It is, genuinely, unlike anything else.

---

*"We believe in fast and secure messaging that's available to everyone."* — Telegram's founding ethos

*"We also believe in not having more than thirty employees, apparently."* — Everyone else in Silicon Valley 🏖️

---

**See also**: [Pavel Durov — The Man Behind the Machine](../people/pavel-durov.md)

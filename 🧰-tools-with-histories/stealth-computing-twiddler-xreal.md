# Stealth Computing With Twiddler 4 And XREAL One Pro 🥷

A workstation usually announces itself with a screen on a desk and a keyboard underneath. Pair a Twiddler 4 handheld keyboard with XREAL One Pro display glasses and a compatible host device, and those familiar parts can travel around a person instead.

Here, **stealth computing** means using a computer with a small visible footprint: a display worn as glasses, discreet handheld input and a host device providing the applications. The appeal is being able to work without first unfolding a little office.

## 🧩 The Workstation Comes Apart

There are three distinct jobs:

| Part | Its job |
|---|---|
| Twiddler 4 | Enter text and control the pointer |
| Host phone, tablet or computer | Run applications and receive input |
| XREAL One Pro | Display the host’s output in front of the wearer |

In a typical arrangement, the Twiddler connects to the host by Bluetooth, while the glasses connect to its video output. The keyboard and glasses cooperate through that host; they do not need to pair directly with each other. [Twiddler connectivity](https://www.tekgear.com/twiddler-4-wrap.html), [XREAL connection guide](https://tutorials.xreal.com/docs/glasses/one-series/first-use/connect-device/)

A phone-based arrangement and a laptop-based arrangement can share this architecture while offering quite different application and window behaviour.

## 👓 What The Glasses Bring

The One Pro’s specification lists a 57-degree field of view, 1920 × 1080 display resolution, refresh rates up to 120 hertz, and a weight of 87 grams. Field of view describes the angular span of the displayed image; hertz describes how many times per second the display refreshes. [Official One Pro specifications](https://tutorials.xreal.com/docs/glasses/one-series/spec/)

Its X1 chip supports native **three degrees of freedom**, or 3DoF: tracking head rotation around three axes. That supports spatial display behaviour as you turn your head. Tracking rotation alone does not provide full positional tracking as you walk around a room. [One Pro product information](https://www.xreal.com/one-pro)

The advertised 171-inch screen is a perceived size at a stated virtual distance of four metres. It does not mean the glasses contain a television panel that somehow escaped the laws of luggage. Screen size and distance are adjustable presentation settings. [XREAL’s display adjustment guide](https://tutorials.xreal.com/docs/glasses/one-series/osd/dispaly-adjust/)

For text work, the useful target is readable characters and comfortable access to the edges. Enlarging the apparent image does not manufacture more pixels. A modest, legible workspace may serve a paragraph better than a cinema-sized spreadsheet. The profile of [XREAL founder Chi Xu](../👤-people/chi-xu.md) follows the consumer-product ambition behind this approach.

### 🧭 Turning Your Head Is Different From Moving It

Imagine a virtual monitor fixed in front of your chair. Turn your head right and, for the monitor to seem stationary, its image must shift left within your view. Rotation tracking supplies information needed for that adjustment.

Now keep facing forward and move your head sideways. A real monitor would appear from a slightly different angle. Reproducing that change requires tracking position too. Three degrees of freedom describe rotation; six add movement along three spatial axes. This is why “anchored” deserves a closer look: keeping a direction stable and attaching an object convincingly to a place in a room are different achievements. The One Pro’s built-in rotation tracking should be understood in that narrower sense. [XREAL’s description of native tracking](https://www.xreal.com/one-pro)

XREAL offers an optional Eye camera module for six-degree-of-freedom spatial anchoring on the One series. That adds a different tracking capability to the arrangement; it should not be assumed from the glasses’ built-in rotation tracking alone. [XREAL Eye’s tracking description](https://www.xreal.com/eye)

The image also lives inside a limited viewing window. A large virtual workspace does not mean its entire area is visible sharply at once. Think of looking at a mural through a frame: enlarging the mural and enlarging the frame solve different problems.

For reading, distinguish two adjustments. Increasing application text size gives each character more of the available pixels, usually leaving room for fewer characters. Increasing the virtual screen’s apparent size changes how large the image looks. Neither creates additional source detail. That is why a spectacular diagonal measurement tells us surprisingly little about comfortable editing.

## 🔌 The Host Is The Quietly Important Part

A USB-C connector (the reversible oval plug) is not enough by itself. Direct connection requires the host port to support **DisplayPort** video output and provide power. XREAL’s guide specifies both requirements and describes adapters for other video sources. [Connection requirements](https://tutorials.xreal.com/docs/glasses/one-series/first-use/connect-device/)

The host also determines whether applications mirror a phone screen, use a desktop-style environment, or offer external-display features. Charging while using the glasses introduces another practical question: whether the particular adapter or hub supports both the required video path and power delivery.

An optional extension is a remote computer doing the heavy work. The local host can act as the terminal while a machine elsewhere runs the demanding applications. The cabinet’s [remote development article](remote-development-anywhere.md) explores that arrangement.

## ✍️ Why The Twiddler Completes The Idea

A head-worn screen can make reading portable. Writing still needs a usable route back into the machine.

The [Twiddler’s chord approach](twiddler-4.md) offers text entry in one hand, with pointer control alongside it. That makes the whole editing loop worth considering: type, correct, select, navigate and return to the sentence. Convenient access to Backspace, Escape and application shortcuts can matter as much as raw typing speed.

There is also an older lineage here. [Thad Starner](../👤-people/thad-starner.md) has described wearing a computer with a head-up display in daily life since 1993. Today’s arrangement revisits that wearable-computing ambition with different hardware and a consumer display ecosystem. [Starner’s biography](https://faculty.cc.gatech.edu/~thad/01_cv.htm)

## 📨 Follow One Sentence Through The System

Consider a hypothetical seated session: open a document, write a sentence, correct a word and switch to a reference page. The Twiddler sends input to the host. The host decides which application receives it and updates the document. Its video output carries the resulting image to the glasses. The wearer sees the result and makes the next correction.

That route explains several otherwise puzzling failures. A pointer that moves while letters go nowhere may indicate that the wrong window has keyboard focus, meaning it is the one receiving typed input. A working keyboard with a blank display says little about whether the host is sending compatible video. A readable phone interface can still be awkward if the application expects touch gestures that have no convenient keyboard equivalent.

These are diagnostic examples, not reports from a tested combination. The architecture shows what must cooperate; it cannot certify every phone, adapter and application.

It also changes what counts as a successful demonstration. Showing a desktop establishes that an image can arrive. Completing the sentence, fixing the mistake and returning from the reference page establishes much more about whether the arrangement can support work. The unglamorous correction is where a portable workstation starts earning its name.

## 🫥 Small Footprint, Real Tradeoffs

The visible screen is less exposed than an open laptop, but privacy still depends on the rest of the system: host-screen mirroring, notifications, audio and the applications themselves. “Stealth” describes the physical arrangement; it is not a security property.

The interesting social tradeoff is that a less visible interface can also make your attention harder for other people to read. An open laptop clearly announces work. Glasses and a small controller offer fewer clues about whether you are available for conversation.

And the desk has not disappeared from physics entirely. Its jobs have moved into a cable, a battery, a learned finger vocabulary and the fit of the glasses. What makes the combination compelling is that those pieces can be carried and positioned separately. The workstation starts to behave more like something you wear than somewhere you sit.

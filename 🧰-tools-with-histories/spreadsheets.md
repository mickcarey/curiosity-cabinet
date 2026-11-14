# Spreadsheets: The Calculator Became A Canvas 🧰

Suppose you are organising a small concert. Eighty tickets at $15 each should bring in $1,200. The room costs $600, and refreshments cost $3 per guest. So far, so manageable. Then someone asks: what if only fifty people come? What if the room costs more? Could cheaper tickets attract enough people to make up the difference?

A calculator can answer each question. A spreadsheet can hold the relationships between them, so changing one assumption sends the consequences through the whole model.

That is the interesting invention hiding beneath the grid. A **spreadsheet** is a sheet of addressable boxes, called cells, which can contain labels, values or formulas. A formula can use values from other cells. The sheet therefore remembers how an answer was made, and can make it again when the ingredients change.

## 🕰️ Tool Biography: Making “What If?” Cheap

On paper, changing a financial model meant recalculating by hand. One number changed, and the whole sheet sighed dramatically.

VisiCalc, created by Dan Bricklin and Bob Frankston and released for the Apple II in 1979, made the spreadsheet interactive. Change an assumption, recalculate the model, ask "what if?" without rebuilding the whole temple. The Smithsonian preserves its software and documentation, while the Computer History Museum describes how its usefulness helped sell Apple II computers. [Smithsonian collection](https://www.si.edu/object/software-and-documentation-software-arts-inc-visicalc%3Anmah_1696121), [Computer History Museum’s 1979 timeline](https://www.computerhistory.org/timeline/1979/)

Bricklin was studying at Harvard Business School when he developed the idea. Business cases supplied a particularly good problem: you wanted to examine the consequences of different assumptions, but updating the arithmetic consumed time that could have gone into understanding the business. [Harvard’s account of Bricklin and VisiCalc](https://www.hbs.edu/about/campus-and-culture/campus-built-on-philanthropy/indoor-spaces/bricklin-classroom)

The distinction matters. Faster arithmetic saves labour on a question you already intended to ask. Cheap recalculation also makes it worthwhile to ask more questions. You can try a pessimistic attendance estimate, adjust a price, notice an unexpected sensitivity and follow it. The model becomes something to think *with*.

### 🧪 The Prototype That Could Not Yet Divide

Bricklin’s own history includes a lovely detail from an early classroom use. He used a developing version of VisiCalc for a Pepsi-Cola case, then typed up the results on a typewriter. When his approach was questioned, part of the explanation he kept to himself was that division was not working yet. [Bricklin’s account of the early days](https://bricklin.com/history/saiearly.htm)

It is a useful antidote to the polished invention story. The future of business computing was already helping with homework while still missing a rather important piece of arithmetic.

## 🎟️ A Small Model You Can Follow All The Way Through

Return to the concert. Column A can hold descriptions; column B can hold inputs and calculations. A cell address such as `B2` means column B, row 2. In a modern spreadsheet, a leading equals sign tells the application to interpret an entry as a formula.

| Cell | Meaning | Entry | Result |
|---|---|---|---|
| B2 | Number of guests | `80` | 80 |
| B3 | Ticket price | `15` | $15 |
| B4 | Room hire | `600` | $600 |
| B5 | Refreshment cost per guest | `3` | $3 |
| B7 | Ticket revenue | `=B2*B3` | $1,200 |
| B8 | Refreshment bill | `=B2*B5` | $240 |
| B9 | Money left after these costs | `=B7-B4-B8` | $360 |

The asterisk means multiplication. These are ordinary arithmetic relationships, written once and connected to the cells containing the assumptions. [Microsoft’s explanation of formulas and references](https://support.microsoft.com/en-gb/excel/get-started/overview-of-formulas-in-excel)

Change attendance in `B2` from 80 to 50. Revenue falls to $750, refreshments to $150, and the remaining money becomes zero. The room still costs $600: it is a **fixed cost** in this model. Refreshments vary with attendance.

Each additional guest contributes $12 towards the room, because $15 comes in and $3 goes out. Dividing $600 by $12 gives fifty guests to cover the modelled costs. That is the **break-even point**. The spreadsheet lets you discover the relationship experimentally; the explanation tells you why it happens.

Now reduce the ticket price to $12. Each guest contributes $9, so covering the room requires 66⅔ guests, or at least 67 whole people. The sheet can calculate this consequence. It cannot establish whether the cheaper ticket will actually attract those extra people.

That boundary is crucial: a spreadsheet follows the assumptions you give it. Here we have deliberately left out staffing, equipment, refunds and unsold refreshments. A correct formula can produce a misleading forecast when its little world is missing something important.

## 🧵 What Happens When One Cell Changes?

There is a second structure underneath the visible rows and columns: a network of dependencies. In the concert sheet, attendance feeds revenue and refreshments; both feed the remaining-money calculation.

Excel tracks these relationships and builds a calculation order. When an input changes, dependent cells are marked for recalculation, including cells affected indirectly. It can update the relevant chain instead of blindly treating every cell as unrelated. Microsoft calls these cells “dirty”, which here means their answers may be out of date, not that someone spilled coffee on them. [Excel’s recalculation machinery](https://learn.microsoft.com/en-us/office/client-developer/excel/excel-recalculation)

The final answer could sit above its inputs on the screen. Calculation follows the relationships, not the page layout.

A **circular reference** appears when a formula depends on itself, directly or through other cells. Imagine two boxes saying “take the other box’s value and add one”. There is no pair of numbers that satisfies both. Other circular models can have meaningful solutions through repeated calculation, but repetition alone does not guarantee a sensible answer. Excel normally warns about circular references and provides optional iterative calculation for deliberate cases. [Circular dependencies and iteration](https://learn.microsoft.com/en-us/office/client-developer/excel/excel-recalculation)

## 🧩 How Copying Becomes A Tiny Programming Language

A spreadsheet becomes much more powerful when you repeat a relationship. Suppose one row describes Friday’s concert, the next Saturday’s, and each row has attendance in column B and ticket price in column C.

Put `=B2*C2` in `D2`, then copy it down one row. With ordinary **relative references**, it becomes `=B3*C3`. The copied instruction means “multiply the two values in this row”. You have expressed a reusable rule without writing a loop.

Sometimes that movement is exactly wrong. Suppose every row should use one shared price stored in `F1`. Writing `=B2*$F$1` lets the attendance reference move when copied while keeping the price reference fixed. The dollar signs mean “keep this column and row”, not “this is money”. Such a reference is called **absolute**. [Relative, absolute and mixed references](https://support.microsoft.com/en-us/excel/switch-between-relative-absolute-and-mixed-references)

This small distinction explains both the appeal and a familiar class of mistake. Copying can spread a correct rule across hundreds of rows, or spread a subtle error just as efficiently. The grid makes repetition easy; the person still needs to decide what should stay the same.

## 📈 From VisiCalc To A Whole Working Environment

VisiCalc made a compelling case for interactive modelling. Lotus 1-2-3, released in 1983, combined spreadsheet work with charting and database functions. The three jobs belonged together: collect figures, calculate with them, then present a pattern someone else could understand. [The National Archives’ format history](https://www.nationalarchives.gov.uk/PRONOM/Format/proFormatSearch.aspx?id=169&status=detailReport), [Computer History Museum’s discussion with the creators](https://computerhistory.org/events/origins-impact-visicalc/)

Microsoft Excel arrived on the Macintosh in 1985; its Windows version followed in 1987. The order is a nice historical wrinkle for software so closely associated with Windows. [Microsoft’s 1985 history](https://learn.microsoft.com/en-us/shows/history/history-of-microsoft-1985), [Microsoft’s 1987 history](https://learn.microsoft.com/en-us/shows/history/history-of-microsoft-1987)

What makes this progression interesting is the expanding boundary of the document. A worksheet can hold raw observations, assumptions, calculations and an explanation of the result. Add a chart and it becomes a presentation. Add a repeated procedure and it starts behaving like an application. The familiar surface can remain a grid throughout.

## 🏗️ Why It Mattered

- It gave personal computers a serious business reason to exist.
- It turned modelling into conversation.
- It made non-programmers capable of building small software-shaped tools.
- It quietly created a world full of mission-critical cells named `B17`.

Consider how a hypothetical volunteer group might use the concert sheet. Initially, one organiser wants to see whether a room is affordable. Next, someone adds actual ticket sales. Then another person adds suppliers, payments and a second event. Eventually the sheet is both the plan and the record of what happened.

No single addition looks like “building a software system”. Yet the group now depends on the file, its formulas and the knowledge of whoever understands its odd corners. The person who knows why column H must never be touched has become part of the machinery.

This is a plausible route from experiment to infrastructure, and it explains why dismissing spreadsheets as amateur programming misses something. The people closest to a problem can build a useful tool immediately. The harder question is how that tool should change as more people rely on it.

## 🪤 The Strange Bit: A Document That Executes

Spreadsheets are both documents and programs. That is their superpower and their curse. They invite experimentation, then sometimes become load-bearing infrastructure wearing office clothes.

A document invites you to edit a number. A program needs you to distinguish between an input and a calculated result. In the same grid, those can look nearly identical.

Suppose someone replaces the formula in the concert’s revenue cell with `1200`, because that is the answer they want to display. Nothing looks wrong at eighty guests. Change attendance to fifty, however, and revenue stays frozen while refreshments fall. The sheet now reports $450 remaining instead of zero. It has become a mixture of live reasoning and a fossilised answer.

The problem is visible once you know where to look. Separating inputs from calculations, giving assumptions clear labels and checking simple cases makes the reasoning easier to inspect. At zero guests, our model should show a $600 loss. At fifty, it should break even. These checks test the intended relationships rather than merely admiring a plausible-looking total.

There is also a difference between a modelling canvas and a shared record system. If several people must update bookings, prevent duplicate tickets and preserve a reliable history of changes, the important questions concern identity, permissions and consistency as well as arithmetic. A bigger grid does not, by itself, answer them.

### 📅 The Day That Never Happened

Excel’s 1900 date system preserves a peculiar historical mistake: it treats 1900 as a leap year, although the Gregorian calendar does not. Microsoft explains that Lotus 1-2-3 made this assumption, and Microsoft’s spreadsheet products retained it for compatibility.

Spreadsheet dates are represented internally by serial numbers. Removing the imaginary day would change how existing numbers map to dates, disrupting established files and calculations. Microsoft therefore documents the behaviour rather than simply correcting it. [Microsoft’s explanation of the 1900 leap-year issue](https://learn.microsoft.com/en-us/troubleshoot/microsoft-365-apps/excel/wrongly-assumes-1900-is-leap-year)

A calendar error became part of a compatibility contract. That is a remarkably compact fossil: the history of competing software products, preserved inside a date cell.

## 🌱 Why It Is Interesting

The spreadsheet is one of the most democratic programming environments ever made. It does not ask you to become a software engineer first. It just hands you a grid and whispers, "Try a formula."

Its deeper trick is keeping an idea and its consequences close together. You can see an assumption, change it and watch the result without leaving the page. That makes the spreadsheet a thinking tool as much as a calculating one, a connection worth following into [Douglas Engelbart’s ideas about tools that extend thought](../👤-people/douglas-engelbart.md).

The concert model’s most useful result might not be its final dollar amount. It might be the moment someone notices that room hire dominates the budget, or that their whole plan depends on an attendance guess. The grid has helped turn a vague concern into a relationship that can be examined.

The formulas can make the arithmetic dependable. Understanding what belongs in them remains wonderfully, inconveniently human.

## 📚 Follow-Up Trails

- [Computer History Museum: The Origins and Impact of VisiCalc](https://computerhistory.org/events/origins-impact-visicalc/), a discussion featuring Bricklin, Frankston and Lotus founder Mitch Kapor.
- [Dan Bricklin: VisiCalc Information](https://www.bricklin.com/visicalc.htm), the co-creator’s collection of history and original material.

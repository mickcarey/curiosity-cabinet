# How Cells Read And Rewrite The Library 📖

Having a library is nothing if nobody reads it. Having a library that *cannot* be rewritten is a museum. Life is neither. Life is a reading room with a printing press in the basement and, every so often, a virus that reverse-photocopies the bootleg back onto the shelves.

[Chapter 03](03-dna-the-library-that-builds-the-librarian.md) was the molecule, the suitcase, the forty-six books, the second score in the mitochondria. This one is the verbs: transcribe, translate, copy a body, make a gamete, silence a page, steal a page, break a letter and sometimes invent jazz.

A body does all of this before breakfast whether anyone is watching. This is not medical advice. The GP is not a ribosome.

## The Dogma And The Leaks 🚰

Francis Crick's **central dogma**, 1958, was a narrower, sharper claim than the poster:

> Once information has got into a protein, it can't get out again.

Sequence information flows among nucleic acids, and from nucleic acids into protein. It does not flow from protein back into a gene. That is the wall. Crick allowed DNA → DNA, DNA → RNA, RNA → protein, RNA → RNA, and even, as a dotted line, RNA → DNA. He did not allow protein → DNA. Lamarck does not get a polymerase.

School shortened this to **DNA → RNA → protein**, which is Watson's textbook version, and then treated reverse transcription as a scandal that "overturned the dogma." It overturned the poster. Crick was, on this point, annoyingly right. Howard Temin and David Baltimore found **reverse transcriptase** in 1970: an enzyme that copies RNA into DNA. Retroviruses use it to paste themselves into a host genome. *Nature* ran an editorial called "Central dogma reversed." Crick wrote back, in so many words: that was never the dogma.

The leaks that actually matter:

- **Reverse transcriptase.** Viral, yes. Also cellular: **telomerase** is a reverse transcriptase with its own RNA template, adding TTAGGG caps to chromosome ends. The library's endpapers are maintained by a leak. **LINE-1** elements, those genome-scale squatters from chapter 03, copy themselves through an RNA intermediate and reverse-transcribe back into DNA. Half your genome is a monument to this leak.
- **RNA viruses** that copy RNA from RNA, never bothering with DNA.
- **RNA editing**, which changes letters in the message after it has been transcribed, so the protein is not quite what the gene said.
- Still, stubbornly, **no protein → nucleic acid**. Prions template shape onto shape. That is heredity in origami, from [chapter 01](01-what-even-is-life.md), and it still does not write a gene.

The dogma is a one-way door on *sequence*. Regulation, splicing, editing, and epigenetics are all ways of performing the score without reprinting it. Keep the door. Enjoy the leaks.

## Transcription: Photocopying A Page 🎤

A gene, for this paragraph, is a stretch of DNA that gets copied into RNA. **RNA polymerase** unzips a local bubble and builds an RNA strand, 5′ to 3′, U where DNA would have put T. The DNA zips up behind it. The gene is not consumed. You can transcribe it again, like a chart a band can play every night.

Humans keep three nuclear RNA polymerases, because of course we do. **Pol II** makes mRNA and many noncoding RNAs, the headliner. **Pol I** makes the big ribosomal RNAs. **Pol III** makes tRNA and 5S rRNA, the small essential parts. Different rooms, different microphones, same library.

In bacteria this is relatively direct: one polymerase, a promoter, a terminator, a message that can start being translated before it has even finished being written. In your cells it is a studio session. Transcription happens in the nucleus. Translation happens outside it. The message has to graduate.

- A **promoter** is where the polymerase and its friends sit down. **Transcription factors** are the producers: they bind DNA and say "this one, now, in this cell."
- The first RNA is a **pre-mRNA** stuffed with **introns** (the bits that will be cut out) and **exons** (the bits that will be kept). **Splicing** removes the introns. The spliceosome is another ancient RNA-protein machine. Most human protein-coding genes are mosaics. **Alternative splicing** lets one gene yield several proteins, like a guitarist reading the same chart as a ballad or a rumble.
- A cap on the 5′ end, a poly-A tail on the 3′, and the message can leave the nucleus.

GENCODE currently lists on the order of 19,000 to 20,000 human protein-coding genes and many more transcripts. The genome is not a list of proteins. It is a list of performances.

Three RNA careers, before we hit the ribosome:

- **mRNA** — the message, the photocopy of a page
- **tRNA** — the adaptor, one end a codon-complement, the other an amino acid
- **rRNA** — the bulk of the ribosome, the oldest instrument in the pit

Most of the RNA in a cell is not mRNA. The factory floor is ribosomal.

## Translation: The Ancient RNA Machine ⚙️

The genetic code is triplets. Sixty-four codons, twenty amino acids plus stops, with redundancy. **AUG** is usually the start, and methionine's foot in the door. Three stops halt the chain. The third position of a codon is often a shrug: **wobble**, Crick's word, meaning the last letter can change and still call the same amino acid. That is why a random typo is not always a plot twist. The table was built, or frozen, or both, to be drunk-proof.

tRNA is Crick's adaptor made flesh: an anticodon at one end, a charged amino acid at the other, a matching done by aminoacyl-tRNA synthetases that are, themselves, some of the oldest protein families we have. Get the charging wrong and the ribosome will still trust you. The ribosome checks the pairing, not the cargo. The synthetases are the real bouncers.

The **ribosome** reads the mRNA, matches tRNAs, and stitches a peptide bond. It has three tRNA seats — A, P, E — like a conveyor: arrive, peptide, exit. In eukaryotes a small subunit finds the cap and scans for the start; in bacteria it lands near a Shine–Dalgarno sequence. Same sport, different kickoff. It is mostly RNA by mass and, at the business end, RNA by catalysis. Crystal structures around 2000 — Ban, Nissen, Steitz, Moore, and the rest of that heroic pile of ribosome labs — showed the **peptidyl transferase centre** is a ribozyme. No protein side chain sits close enough to do the chemistry. The proteins are scaffolding, quality control, the roadies. The bond is an RNA trick.

That is the weirder-than-school beat, and it is the origin-of-life beat wearing a lab coat. If the machine that makes protein is made of RNA, RNA did not need protein to invent protein. The ribosome is a fossil that still has a day job. LUCA, from [chapter 01](01-what-even-is-life.md), already had one. It never unionised. It never left.

Translation is slow, expensive, and accurate enough. A calorie, later in this series, is partly the bill for this. Making a protein is not free. Cells do not translate everything they transcribe, and they do not transcribe everything they could. The library is full. The reading list is short.

## The Octopus Remix 🐙

Most animals edit RNA a little. Coleoid cephalopods — octopus, squid, cuttlefish — edit RNA like it is a living.

The usual trick is **ADAR** enzymes turning adenosine into inosine. Inosine is read as guanosine. An A in the gene becomes a G in the message. The protein changes without a mutation in the DNA. In humans this is rare in coding sequence, common in noncoding RNA, and easy to oversell. In coleoid brains it is a lifestyle. Liscovitch-Brauer, Rosenthal, Eisenberg and colleagues, in *Cell* in 2017, found **tens of thousands** of conserved recoding sites in octopus, squid, and cuttlefish, concentrated in neural transcripts. Nautilus, the less baroque cephalopod, does not play this sport.

There is a trade-off. ADAR needs double-stranded RNA structure around the site. If you mutate those flanking sequences, you lose the edit. So the genome around recoding sites is unusually conservative. Cephalopods appear to have **traded genomic evolution for transcriptome plasticity**: fewer DNA changes, more live remixes. Temperature can shift the mix. Experience might. We do not fully know the dimmer switches.

The cabinet already keeps a door open on this in [The Mystery of Octopus Intelligence](../✈️-airplane-reading/the-mystery-of-octopus-intelligence.md). Nine brains, three hearts, and a genome that lets the RNA department do jazz. If you want a picture of "the library is not the performance," an octopus is the picture. They are not programmable in the science-fiction sense. They are recodable in the biochemical one. That is stranger, and sourced.

## Mitosis: Copying A Body 👯

**Mitosis** is how a body copies a cell. One diploid cell becomes two diploid cells. The 46 chromosomes are duplicated first (now 46 chromosomes, each as two **sister chromatids** joined at the centromere), lined up, and pulled apart so each daughter gets a matching set. Growth, repair, skin, the lining of a gut that replaces itself like a stadium crowd. The point is **sameness**. A liver cell wants another liver cell, not a surprise.

Homologues do not pair up to swap chapters. Sisters split. The karyotype is preserved. When this goes wrong you get aneuploidy in a somatic lineage, which is a cancer story and a later chapter, not a pregnancy story.

Think of a band photocopying the setlist so the second stage can play the same gig. No new songs. No shuffled order. Just another night.

## Meiosis: Making A Gamete 🎲

**Meiosis** is how a body makes a **gamete**. The point is **half**, plus **shuffle**.

Two divisions.

**Meiosis I** is the weird one. Homologous pairs find each other and pair up along their lengths (**synapsis**), held by a protein zipper called the synaptonemal complex. Then — in **crossing over** — they swap stretches of DNA between non-sister chromatids. You can see the joins as **chiasmata**. Each chromosome that walks away can be a mosaic of maternal and paternal editions, not a clean copy of either. Then the homologues separate: **reduction division**. Each daughter is already haploid for chromosome *number*, though each chromosome may still be two chromatids.

**Meiosis II** looks more like mitosis: sisters split. Four cells. In humans, if all goes well, **23 chromosomes** each. Haploid. Sperm production tends to make four sperm. Egg production is stingier with cytoplasm: usually one egg and polar bodies. The chromosome maths is the same. The catering is not. Pregnancy, embryos, and the rest of the construction site wait for chapter 10.

**Independent assortment**: each of the 23 pairs orients at random. 2²³ is about **8.4 million** combinations before you even count crossing over. Fertilisation multiplies two such lotteries. The unique-person speech writes itself, and then you remember siblings, and then you remember that unique is not the same as inexplicable. It is shuffling. Very good shuffling.

Crossing over is not a bonus round. It is also mechanical. Homologues need those chiasmata to segregate properly. Fail the dance and you get the wrong number in a gamete. **Nondisjunction** is the polite word. Chapter 10 is where fertilisation, embryos, and chromosomal errors as failed dances belong. This chapter stops at the gamete: a haploid cell with a remixed half-library, waiting.

Why 23, not 46? Because fertilisation will add the other 23. Diploidy is a reunion, not a default. Meiosis is the intermission where the band splits into two half-lineups so the next gig can mix them.

Mitosis copies a body. Meiosis makes a ticket to the next generation. Confusing them is how you get a Year 10 worksheet that thinks sperm are "made by mitosis, sort of." They are not.

## Who Gets To Speak: Regulation 🎛️

Every nucleated cell in your body (give or take a few stunts) has the same nuclear library. A neuron is not a hepatocyte because it *lost* the liver genes. It is a neuron because it is **not reading** the liver genes, and is reading a pile of neural ones instead.

**Gene regulation** is the reading list.

- **Transcription factors** bind specific DNA sequences. Combinations matter more than any one celebrity protein. A promoter is a door. An **enhancer** can be a door-opener thousands of bases away, looped in through 3D space. The genome is not a line. It is a folded city.
- Chromatin is the volume knob, as chapter 03 promised. Open nucleosomes, closed nucleosomes, histone tails chemically decorated.
- **Noncoding RNAs** join in: microRNAs that silence messages, long noncoding RNAs that can recruit chromatin machinery, a zoo we are still naming.
- In female mammals, one X is largely silenced (**X-inactivation**), as chapter 03 already flagged: a whole chromosome treated as a volume problem. Dosage compensation is regulation at architectural scale.

The weirder lesson: **most of the genome's interesting work may be deciding when not to speak.** Protein-coding sequence is the album. Regulation is the producer, the mixing desk, and the decision not to release the track. School loved the album. The desk is the career.

A football analogy, because the mixing-desk one will get tired: every cell has the same squad. Differentiation is the manager's sheet. A goalkeeper is not a striker who lost the shooting genes. He is a player whose job sheet says "hands," enforced by which transcription factors showed up to training. Trade the sheet, and you can, in the lab, sometimes talk a cell into a different position. That is the whole of reprogramming, and it is regulation, not a new library.

## Epigenetics: Notes In The Margins 📝

**Epigenetics**, used carefully, means heritable changes in gene activity that are not changes in DNA sequence. "Heritable" here often means **through cell division**, not through your grandchildren. That distinction is where the pop-science bus leaves the road.

The main pencils:

- **DNA methylation**, typically cytosine in CpG contexts, often associated with silencing
- **Histone modifications** — acetyl, methyl, phosphate tags on those tails — a code-ish language that readers (proteins) interpret as open, closed, poised
- **Noncoding RNAs** that can help keep a silent region silent

This is how a stem cell's granddaughter becomes bone and stays bone, memory without rewriting the gene. It is also how the environment can, sometimes, lean on the desk: diet, stress, a toxin. Intergenerational effects (parent to child) are real in some systems. **Transgenerational** epigenetic inheritance — surviving the reset that happens in the mammalian germline — is limited, contested, and much easier to demonstrate in plants and worms than in humans. Mammals reprogram methylation twice per generation. Most notes in the margins get erased. A few marks may escape. Do not build a personality theory on the escapees.

**Imprinting** is the exception that earned a name. A handful of human genes are marked as "from dad" or "from mum" and only one copy is allowed to speak. That is epigenetic, sequence-addressed, and required for the two parental genomes to not shout over each other in an embryo. It is also why you cannot just clone a mammal by wishing two haploid sets together without the marks. Details: chapter 10. Here it is evidence that some margin notes *do* survive into the next generation, on purpose, at specific loci. That is not the same as "your mood methylated your grandchildren."

This is not medical advice, not a parenting manual, and not proof that your grandfather's famine is your destiny. Sequence still sits underneath. Epigenetics is how the same score is performed in different rooms.

## Horizontal Gene Transfer: Passing Notes In Class 📨

Vertical inheritance is parent to child. **Horizontal gene transfer (HGT)** is a neighbour sliding you a page.

Bacteria do this for a living:

- **Transformation** — eat free DNA
- **Transduction** — a virus accidentally packages host DNA and delivers it to the next cell
- **Conjugation** — a pilus, a plasmid, the original USB cable

This is why antibiotic resistance travels faster than a textbook tree of life can draw. The tree is a tree until it is a **web**. Chapter 16 will enjoy that sentence.

Eukaryotes are snobbier and still not innocent. Endosymbiosis was the big one: mitochondria and chloroplasts are bacteria that moved in, and a crowd of their genes subsequently walked into the nucleus (**endosymbiotic gene transfer**). *Agrobacterium* still injects DNA into plants for a living; we stole the trick to make GM crops. Aphids got carotenoid genes from fungi. Humans are not a HGT carnival, but our genomes are half transposon, which is a related kind of trespassing.

CRISPR, next heading, evolved as a way to *remember* the notes you did not want passed.

## CRISPR: Bacteria Invented Adaptive Immunity First 🦠

**CRISPR** is Clustered Regularly Interspaced Short Palindromic Repeats. The name is a dare. The job is a mugshot album.

Bacteria and archaea stash short sequences from viruses that have attacked them, between repeats, in the genome. On the next visit, CRISPR RNAs guide **Cas** nucleases to cut matching DNA. Francisco Mojica noticed the repeats and, by 2005, the viral origin of the spacers. Rodolphe Barrangou and colleagues, in 2007, showed in *Streptococcus thermophilus* — a yogurt bacterium, which is the correct amount of humble — that new spacers appear after phage attack and confer sequence-specific, heritable resistance. Adaptive immunity. In a prokaryote. Before it was a stock ticker.

The lab tool came later: a guide RNA plus Cas9 as programmable scissors, Doudna and Charpentier and a thousand papers. That is a powerful instrument and a public argument. This chapter's point is the original one. CRISPR is not "we invented gene editing." CRISPR is **bacteria keeping a library of their enemies inside the library of themselves**, then using RNA to find the match. Immunity first. Editor second.

It is also, if you like the theme, a leak in the other direction: information from a virus's genome, written into the bacterium's genome, on purpose. Horizontal, adversarial, remembered.

## Mutation: Damage And Engine 🔧

A mutation is a change in the sequence. UV dimers, replication slips, oxidative hits, a polymerase that shrugged, a transposon landing in the sofa. Proofreading and repair catch most of it. They do not catch all of it. Chapter 03's one-in-a-billion is how you still collect a handful of new variants every time a cell divides.

Most mutations are **silent** or **near-silent** (redundancy in the code, junk-ish sequence, a protein that does not care about that amino acid). Some are **bad**. A few are **useful** in a particular environment. Evolution is not a watchmaker and it is not a vandal. It is a filter on a stream of typos.

Two rooms, because mixing them is how people scare themselves:

- **Somatic** mutation happens in a body cell. It can do cancer. It does not go to your children.
- **Germline** mutation happens in the lineage that makes gametes. It is the only kind evolution, in the Darwinian sense, can see.

The immune system cheats the distinction on purpose. **Somatic hypermutation** in antibody genes is mutation as a tool: B cells turn up the error rate on a specific locus, then select the better binders. Damage, aimed, hired. That is not how your liver evolves. It is how a lineage inside you plays Darwin in a week.

Mutation is damage *and* the engine. Without it, meiosis shuffles a frozen deck. With too much of it, the library burns. The error rate is a negotiated truce, probably as old as replication. Viruses often run hotter error rates and evolve like they are on fire. You run cooler because you have more to lose per generation. Same chemistry. Different risk appetite.

## The Dark Genome 🌑

Call the unread majority the **dark genome** if you want a name with a cape. Protein-coding exons are a percent or two. Conserved, likely-selected sequence is perhaps around a tenth. ENCODE-style biochemical activity is most of the rest, depending on your mood and your definition of "activity." Between those numbers lives the argument from chapter 03, now with the lights of regulation waving around in it.

What is in the dark, as far as we can point:

- Enhancers we have not mapped, in cell types we have not assayed
- Noncoding RNAs whose transcripts exist and whose jobs are rumoured
- Transposons that are dead, sleeping, or occasionally a new promoter
- Structural DNA that exists to be bulk, spacing, a nuclear spring
- Noise. Genuine, selected-against-if-it-gets-expensive, noise

GWAS hits for common traits pile up in noncoding sequence. That is a clue that the desk matters more than the album for a lot of human difference. It is not a decoder ring. A variant in an enhancer in one cell type may do nothing in another. A lot of those hits sit in DNA we still cannot name a job for without waving at "regulation." We are very good at sequencing. We are still guessing at a lot of the mixing desk.

There is also a **dark proteome** mood in the literature now: tiny open reading frames, microproteins, things the gene catalogues were not looking for because they were looking for proper-sized novels. Some of those will be real. Some will be noise with a peptide. The dark genome and the dark proteome are the same unread wing, seen from DNA and from protein.

The dark genome is not a mystery because we have not looked. It is a mystery because "function" was never one thing, and the genome was never written for us to read.

## Still Unsolved 🕳️

- **How much regulation are we still guessing at?** Enhancer logic, combinatorial transcription factors, the 3D nucleus. Maps exist. The grammar is incomplete.
- **How much RNA editing matters in humans?** A little, clearly, in some transcripts. Not an octopus. The gap is interesting, not an insult.
- **How far can mammalian epigenetics travel down a lineage?** Intergenerational, sometimes. Transgenerational, limited and loud in the literature relative to the effect sizes.
- **How much HGT is still happening in animals, quietly?** The web is real in microbes. In us it is mostly old news and endosymbiosis. "Mostly" is doing work.
- **Where did the ribosome's RNA heart come from?** RNA world is a theory that this chapter makes more tempting, not a souvenir. Chapter 16 gets the origin fight.
- **What is the dark genome for?** Some of it, nothing. Some of it, we have not been clever enough. Distinguishing those is the actual problem.

## Things To Look At Later 📚

- Crick (1958 / 1970) on the central dogma, and Matthew Cobb's [60 years ago, Francis Crick changed the logic of biology](https://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.2003243) for the poster versus the claim
- Nissen et al. (2000), [The structural basis of ribosome activity in peptide bond synthesis](https://www.science.org/doi/10.1126/science.289.5481.920) — the ribosome is a ribozyme
- Liscovitch-Brauer et al. (2017), [Trade-off between transcriptome plasticity and genome evolution in cephalopods](https://www.cell.com/cell/fulltext/S0092-8674(17)30346-9). *Cell.*
- Barrangou et al. (2007), [CRISPR Provides Acquired Resistance Against Viruses in Prokaryotes](https://www.science.org/doi/10.1126/science.1138140). *Science.* Yogurt, phages, a mugshot album
- The cabinet's [octopus intelligence](../✈️-airplane-reading/the-mystery-of-octopus-intelligence.md) note, now with a mechanism under the wonder
- Genomic imprinting, if you want the rare epigenetic system that is definitely supposed to travel with the gamete
- Next: proteins, the machines that fold, which is what all of this reading is *for*

If chapter 03 was "the instrument comes with sheet music," this one is "the band actually plays, remixes, shuffles the setlist, and sometimes steals a riff from the support act." The library builds the librarian. The librarian, inconveniently, can also edit the library.

Arc 3 is the machines that fold. You have earned the proteins.

This is not medical advice. Mutation, methylation, and CRISPR in a paragraph are not a treatment plan.

---

### Sources

- Crick, F. H. C. (1958). On protein synthesis. *Symp. Soc. Exp. Biol.* The original dogma: once information is in a protein, it cannot get out. Cobb, M. (2017). [60 years ago, Francis Crick changed the logic of biology](https://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.2003243). *PLOS Biology.* Poster version versus Crick's version; reverse transcriptase as a predicted-ish special transfer, not a reversal.
- Temin, H. M., & Mizutani, S. (1970); Baltimore, D. (1970). RNA-dependent DNA polymerase in virions. Reverse transcriptase; Nobel 1975. Telomerase as a cellular reverse transcriptase: Greider, C. W., & Blackburn, E. H. (1985). Identification of a specific telomere terminal transferase activity in *Tetrahymena* extracts. *Cell.*
- Nissen, P. et al. (2000). [The structural basis of ribosome activity in peptide bond synthesis](https://www.science.org/doi/10.1126/science.289.5481.920). *Science, 289*, 920–930. Peptidyl transferase centre is RNA; proteins stay back. See also Cech, T. R. (2000), "The ribosome is a ribozyme," same issue.
- Liscovitch-Brauer, N. et al. (2017). [Trade-off between transcriptome plasticity and genome evolution in cephalopods](https://pmc.ncbi.nlm.nih.gov/articles/PMC5499236/). *Cell, 169*, 191–202. Tens of thousands of conserved A-to-I recoding sites in coleoids; not in nautilus; neural bias.
- Alberts B. et al. *Molecular Biology of the Cell.* Transcription, splicing, translation, mitosis versus meiosis. Independent assortment as 2²³ combinations in humans; crossing over on top.
- Nature Education Scitable. [Mitosis, meiosis, and inheritance](https://www.nature.com/scitable/topicpage/mitosis-meiosis-and-inheritance-476/).
- Barrangou, R. et al. (2007). [CRISPR Provides Acquired Resistance Against Viruses in Prokaryotes](https://www.science.org/doi/10.1126/science.1138140). *Science, 315*, 1709–1712. Mojica et al. (2005) on spacers matching viral genomes, the hypothesis that Barrangou tested.
- Soucy, S. M., Huang, J., & Gogarten, J. P. (2015). [Horizontal gene transfer: building the web of life](https://www.nature.com/articles/nrg3962). *Nature Reviews Genetics.*
- Heard, E., & Martienssen, R. A. (2014). [Transgenerational epigenetic inheritance: myths and mechanisms](https://pmc.ncbi.nlm.nih.gov/articles/PMC4020004/). *Cell.* Mammalian germline reprogramming as a barrier; do not oversell grandparental destiny.
- Fitz-James, M. H., & Cavalli, G. (2022). [Molecular mechanisms of transgenerational epigenetic inheritance](https://www.nature.com/articles/s41576-021-00438-5). *Nature Reviews Genetics.* Intergenerational versus transgenerational; weaker and rarer in mammals than in plants and some invertebrates.
- ENCODE Project Consortium (2012). [An integrated encyclopedia of DNA elements](https://www.nature.com/articles/nature11247). *Nature.* Biochemical maps of the noncoding genome; "function" still disputed, see chapter 03 sources (Graur; Palazzo & Gregory).
- GENCODE human statistics: ~19,000–20,000 protein-coding genes, many more transcripts. Alternative splicing is the multiplier.
- Mutation as both error and evolutionary fuel: standard population genetics; Drake's rules of thumb on genomic mutation rates per generation. Most new mutations are neutral or deleterious; a few are the engine. Somatic hypermutation in antibody genes is the rare case of a cell turning the error rate *up* on purpose. Not a treatment, not a horoscope.
- LINE-1 reverse transcriptase as a cellular (if parasitic) RNA→DNA activity: see Cordaux & Batzer, chapter 03 sources. Telomerase is the respectable cousin.

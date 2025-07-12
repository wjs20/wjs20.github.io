---
layout: post
title:  "Antibody Discovery - A Primer"
date:   2024-02-11 14:42:47 +0000
categories:
---

I hope this post will serve as a reference point for readers of later posts who
feel they need a little more context on the field of antibody discovery. It will
go into the basics of antibodies as molecules and therapeutics, and some of the
core approaches in the field. If you work with antibodies and you are familiar with
all of this feel free to skip!

# What is an antibody?

<figure style="float: right; margin-right: 20px; width: 50%;">
  <img src="/assets/2024-02-11/trastuzumab-fab.jpg" alt="Trastuzumab as a fab" />
  <figcaption style="font-size: small; text-align: center;">Image Source: <a href="https://www.rcsb.org/structure/6BI0">Protein Data Bank</a></figcaption>
</figure>

Antibodies are proteins. They function as a key component in many animals immune
system, helping to detect and eradicate invading organisms or rogue cancerous
cells from the body. What makes antibodies particularly facinating as a class of
proteins is that the set of antibody proteins we are born with are not the same
as the ones we have as adults. In fact, the set of antibodies circulating in
your body right now is probably significantly different from the ones you had a
month ago. Especially if you have caught a bug in the intervening weeks.

Why is does this make them so interesting? Well, the vast majority of proteins in
our bodies are encoded by genes which do not change throughout our life. Their
underlying genetic sequence is set in stone at the moment our parents genomes
recombine to make ours. Antibodies are different. They can evolve and change
*during our lifetime* in response to our environment.

This evolution results in a dizzying number of unique antibody molecules in
typical adult, with estimates ranging from [tens of millions to well into the
billions](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7153836/).
The human body acheives this remarkable feat using a bag of molecular
tricks. For example, it mixes and matches elements from different gene cassetes
resulting in large number of possible combinations. Once these gene combinations
have been formed, the body can introduce additional variation using targeted
mutations to part of the antibody genes.

The upshot of all this variation is that our antibody repertoires are likely to
contain variants that will *recognise almost any novel entity* nature throws at
us. When a new virus emerges that the human immune system has never encountered
before, chances are we have an antibody that will be able to counter it. Not
only that, but our immune system will actually generate new antibodies in
response this new virus, which will be *even better* at recognising and
neutralizing it.

This is by no means an exhaustive account of antibodies, but I hope it gives you
a flavour as to why they are such interesting and important molecules.

# Antibodies as therapeutics

Antibodies have been central to medical advances for centuries. Any time you get
a vaccine for a disease, the result is that your body produces antibodies that recognize some
piece of the disease causing pathogen. These antibodies are what provide us long term immunity to
disease. However with the growth of industrial biotechnology, it was recognized
that one could isolate the gene encoding a particular antibody through molecular
cloning, and mass produce it.

<figure style="float: left; margin-right: 20px; width: 60%;">
  <img src="/assets/2024-02-11/antibody-approval-growth.png" alt="Growth in
  antibody approval rates" />
  <figcaption style="font-size: small; text-align: center;">Image Source: The Antibody Society via <a href="https://www.crbgroup.com/insights/biotechnology/monoclonal-antibody-manufacturing-introduction">via CRB group</a></figcaption>
</figure>

These cloned antibodies (known as recombinant antibodies) have found a wide
variety of uses in the medical field. Antibodies are key to how pregnancy
tests work for example, which use an antibody recognizing human chorionic
gonadotropin - a hormone released in large amounts in pregnant women. The
arguably more transformative application of recombinant antibodies however
is as therapeutics (i.e. drugs). Once produced in large amounts, antibodies
can be injected back into patients to combat disease.

As you can see from the figure on the left the approvals of antibodies as
therapeutics increased dramatically in the 2010s and has continued to rise since
then, with [over 200](https://www.antibodysociety.org/antibody-therapeutics-product-data/)
antibodies approved for therapeutic use or in regulatory review.

So whats behind this impressive adoption? Well there are several attributes of
antibodies that set them apart from the traditional class of molecules used as
therapeutics (small molecules), and which make them ideal drug candidates. For
one thing, their large size means they interact with their targets over a wide
surface area. This means that the interaction is more likely to be unique (as
there are more potential permutations of interacting groups) which solves a key
challenge for thereapeutics - specificity. We want therapeutics to interact with
their target and *nothing else*. This *off-target* interaction as its known, is
a killer of therapeutic programs, and is one of the major sources of safety
issues. Antibody therapeutics promise to reduce this risk.

In addition to the specificity bonus, antibodies come with a set of built-in
immune system engager functions (they are immune proteins after all!). Through
these elements, they are able to recruit and activate immune cells that are able
to then kill the cell the antibody is attached to, a process known as antibody
dependent celullar cytotoxicity (ADCC).

Finally, the specificity of antibody therapeutics can be used to target the
delivery of other drugs to particular locations in the body. Think heat seeking
missile! By conjugating a toxin to an antibody the binds preferentially to
cancerous cell proteins for example, we can limit the damage done to other
tissues in the body.


# How are Antibody therapeutics discovered?

We have learned in previous sections about some of the salient features of
antibodies, and why their popularity as therapeutics has taken off. But what are
some of the ways they are actually discovered in practice. Well, the answer is there are
many! And the makeup of these approaches has been changing over the years. Lets
get into some of them.

The very first class of approaches to discovering therapeutic antibodies
involved injecting an animal (typically a mouse) with your drug target, and
allowing the animal to develop an immune response to the target (including
target specific antibodies). The B cells (the cell type that produces
antibodies) of the animal would then be harvested and screened to identify those
producing target specific antibodies. The hits would be fused with myeloma cells
(an immortalized B cell specialized for antibody production) from which these
antibodies could be expanded and mass produced. This approach is referred to as
[hybridoma technology](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8521504/).

A later development were the so called *in-vitro* techniques (so called to
distinguish them from the *in-vivo* techniques described in the last paragraph).
This involved isolating antibody genes from a collection of human donors,
cloning them into a new vector which can be used to express the antibody protein
on the surface of a bacteriophage particle. The resulting population of antibody
expressing bacteriophage is known as a *library*. The foundational methods using
this approach were therefore named [phage display](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3656071/).

<figure style="float: right; margin-right: 20px; width: 50%;">
  <img src="/assets/2024-02-11/phage-display-fishing-analogy.jpg" alt="Your Image Description" />
  <figcaption style="font-size: small; text-align: center;">Image Source: <a href="https://www.sciencedirect.com/science/article/abs/pii/S016561471830230X">Trends in Pharmacological science</a></figcaption>
</figure>

The process of extracting target specific antibodies from antibody libraries is
often refered to as biopanning or selection. A good analogy for how this works
is angling. In this analogy our antibody library would represent a body of water
containing fish (antibodies). We attach bait (our drug target) to a hook (biotin) and throw
it into the water. After waiting a sufficient amount of time for fish to bite,
we retract our baited hook using our fishing line (magnetic bead conjugated
streptavidin) and land a fish (antibodies). This is obviously done millions of
times at a microscopic scale, but the principle is remarkably similar!

These two approaches each come with their own strengths, weaknesses and ethical
issues. For example, antibodies discovered using mice have to undergo a process
known as "humanization" so that they can more safely be injected into humans
without eliciting their own immune response, a process which is costly and prone
to failure. However antibodies discovered using phage display do not go through
the same kind of quality checks that occur inside the body during an immune
response, and are therefore thought to be more prone to issues that prevent
their ascension to approved therapeutic, often termed developability issues in
the industry.

New variants of these techniques are being developed to address these
shortcomings. For example, new transgenic mice have been developed that express
human antibody genes, obviating the need for humanization. Similarly, new
variants of in-vitro techniques have emerged which promise to minimize some of
their associated issues. [Mammalian
display](https://pubmed.ncbi.nlm.nih.gov/19252852/) was developed by some of my old
colleagues at Iontas is an effort to reintroduce some of the intrinsic quality
filters present in in-vivo methods.

A third and much newer approach to discovering antibody therapeutics is the
fully *in-silico* approach, where computational techniques are used to design
new antibodies *de-novo*. The kinds of techniques employed here are diverse and growing.
One example generates populations of antibody structures using [combinations of
existing loop fragments](http://www-vendruscolo.ch.cam.ac.uk/sormanni2018csr.pdf). These structures are then docked against the target and
scored using an energy function. Successful designs are subjected to mutagenesis
in an effort to improve binding energy. In-silico approaches are definately in
their infancy (and will surely be the topic of many future blog posts).
Difficulties remain around the ability to accurately model key components of
antibody structures (namely the complementarity determining regions), limiting
the applicability of these approaches. However the arrival of powerful new
machine learning algorithms, combined with the ever increasing size of
structural data available to learn from may ultimately knock down these barriers
completely.

# Conclusion

This post has glossed over many important and interesting facets of antibodies,
however it has hopefully given you an overview of the field that will enable you
to engage with some of the other content in this blog more easily. I also hope I
have convinced you that antibody therapeutics hold huge promise! For those who
want to learn more, I have included some relevant resources below.

- [The Antibody Society](https://www.antibodysociety.org/) website is a good
  place for news and resources relevant to those involved in antibody discovery
- [mAbs](https://www.tandfonline.com/journals/kmab20) is a well respected
  journal in the antibody field. Check out their *most read* section for helpful
  reviews
- [Janeways Immunobiology](https://www.amazon.co.uk/Janeways-Immunobiology-Kenneth-M-Murphy/dp/0393884910/ref=asc_df_0393884910/?tag=googshopuk-21&linkCode=df0&hvadid=606781675864&hvpos=&hvnetw=g&hvrand=16464619030823393981&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9044909&hvtargid=pla-1596925236390&psc=1&mcid=8a770507dbd63d43a1f3ba15da7c215c&th=1&psc=1)
is the preeminent textbook in the field of immunology and has a good section an
antibodies specifically
- [Antibody Drug
  Discovery](https://www.amazon.co.uk/Antibody-Discovery-Molecular-Medicinal-Chemistry/dp/1848166281)
  is a more focused text, and goes into much more detail on many of the topics
  discussed in this post.

---
title: 'Digital Gardens'
date: 2020-07-17 00:00:00
excerpt: Can you dig it?
categories: [lifestyle]
tags: []
featured_image: 'images/demo/demo-portrait.jpg'
#scroll_image:
comments: true
share: true
---

I found out about [Roam Research](https://roamresearch.com/) a few weeks back (on Hacker News, I think.) I fell into its rabbit hole for a little bit and immediately recognized that I had been seeking for a better way to express and connect my thoughts. I have been using [Evernote](https://evernote.com/) for the longest time. It gets the job done. Just so long as the job is "archive this thought somewhere so that I don't forget it forever". I don't think that Evernote, as a tool, has actually helped to increase the depth, breadth, or richness of my thoughts.  

---
# Roaming Your Thoughts

```
roam      
/rōm/    
_verb_  
Move about or travel aimlessly or unsystematically, especially over a wide area.
```
The ability to traverse your notes as if they were your second brain requires the notes themselves to provide the pathways and networks to reach other notes. If I am reading my `TODO` list and I see that I need to shop for groceries, wouldn't it be nice for the note itself to contain a link to my `Shopping List`? And when I do look at my `Shopping List` and I see that I need flour and yeast, wouldn't it be nice to have a link to my `Recipes` to view the no-knead, cold fermented bread recipe I've been meaning to bake? This is a very simple and practical example - most people won't forget that chain of causation and relationships between the notes and their content. But imagine a network with hundreds or thousands of `Notes`, each with multiple `[Links]` pointing to different `Notes`. By having the notes themselves contain the pathways to other thoughts/notes, we can build a network that makes it easier to explore, visualize, and create relationships between certain ideas. Our brains do this inherintely, but we can augment this ability with [bi-directional linking.](https://maggieappleton.com/bidirectionals)

Here's how bi-directional linking works in the contextual nutshell of note-taking:  

1. You are writing a note in Markdown and you make a wiki link to `[[my awesome page]]`.  

2. If a note for `[[my awesome page]]` already exists, `[[my awesome page]]` will be automagically updated at the bottom with something that says "Pages that link to my awesome page: `[[original note]]`  

3. If note for `[[my awesome page]]` does not exist, then it will be created and have same link infoformation applied automagically.  

    
And thus, each page contains information about all of the pages that it links _to_ __and__ all of the pages that link *to it*. By continuing to create notes (nodes) and link them to others (edges) we build a graph that we can traverse and visualize - huzzah! This is basically a note-based version of one of my all-time favorite pieces of software, [Inspiration 9.](https://www.inspiration-at.com/)

### Bi-Directional Note Applications

- [Roam Research](https://roamresearch.com/)
    - Super trendy right now, especially in academia
    - SaaS
- [Obsidian](https://obsidian.md/)
    - Same idea, but runs locally and you can plug into git 
- [Foam](https://github.com/foambubble/foam)
    - Same-same idea, but does it all in VS Code and git

---
# Digital Gardens

My immediate thought with graph-based notes was that I'd like to be able to shove the entire network into my blog. It should be no surprise that there are plenty of brilliant people who are way ahead of me 😄 Not only are there excellent blogs that use the underlying technology, but there is an entire philosophy and subculture behind _"digital gardening"_.

I stumbled into digital gardening through the backdoor. I started with the technology because it was a natural (if rudimentary) augmentation of how we already think and process thoughts - aside from [neural networks](https://en.wikipedia.org/wiki/Artificial_neural_network), I'm not familiar with other examples of [biomimicry](https://biomimicry.org/what-is-biomimicry/) in coding/technology. I imagine that the front door to the garden starts with a desire to grow things and produce better ideas. I'll leave it as an exercise for the reader to figure out what it's all about, but just remember that it's Joe Garden; [it's Joe Dirt. ](https://www.youtube.com/watch?v=FvP7nZQs-nA)

### What Are You Growing? 

+ [You and your mind garden](https://nesslabs.com/mind-garden)
  + An essay about digital gardening.
+ [Maggie Appleton](https://maggieappleton.com/garden/)
  + A crisp, clean, modern digital garden.
+ [Andy Matuschak's Working Notes](https://notes.andymatuschak.org/About_these_notes)
  + One of the most fun and rewarding UI experiences for rambling through notes.
  + These notes are Excalibur-tier legend amongst digital gardeners from what I can gather.
    + I believe that Andy [let a few people use his framework](https://notes.mxstbr.com/About_these_notes) and it has spawned at least [one Gatsby theme](https://github.com/aravindballa/gatsby-theme-andy) since then.
+ [Maxime Vaillancourt](https://maximevaillancourt.com/)
  + A blend of traditional blogging and digital gardening - check out the **Notes** section to see what's growing.
    + Maxime's site includes a graph visualization. 
  + Maxime also has a **Side Project** for a [digital garden jekyll template.](https://github.com/maximevaillancourt/digital-garden-jekyll-template)  


---

# Agriculture is the Mother of Invention

Here is a fun story: While trying to write this article, I had a full-blown digital gardening experience. What you're about to read is a short story about a few minutes of research that would make a perfect snippet or micro-blog (AKA a "seedling".)

---
I was looking for a subtitle for the next section of my blog article where I talked about what components are missing from the tech stack for a really comprehensive, smooth blogging experience that would use graph-based notes. In the back of my mind, I thought I remembered some quote about _"War is the mother of all invention."_ and a vague recollection of "..and that's where microwaves come from, kids!"

So I went with **"Agrilculture is the Mother of Invention."** because, ya know, puns. I did a quick Google just to cover my bases and I found out that ["Necessity is the mother of invention."](https://en.wikipedia.org/wiki/Necessity_is_the_mother_of_invention) But this is where it get's good - Ester Boserup used the phrase as a major argument in her book _The Conditions of Agricultural Growth: The Economics of Agrarian Change under Population Pressure._ So, this just goes to show you that my subtitle for this section is about 9,000 IQ. 

---

I'm a big fan of [quotes](https://www.goodreads.com/quotes/7502834-i-shall-pass-this-way-but-once-any-good-that), so I would probably tag the above snippet with that label and link it to/from this article. It would be so much better if I had actually done that instead of talking about it hypothetically, but that brings me to my next point: there are some tools missing from the toolkit.

The key magic behind graph-based note applications is that they automagically create a new note for you when you type out a link. The note exists and it even contains some text; the backlink to the note that originally linked to it. However, we need a little more than that. If I want my new note to appear as an article of sorts on my blog with minimal effort, I need a little more boilerplate text added than just the title of the note and a backlink. At minimum, we'd need to add the Markdown header (if you're using a Markdown based website.) Here is an example of the Markdown header for this article:

```
---
title: 'Digital Gardens'
date: 2020-07-17 00:00:00
excerpt: Can you dig it?
categories: [lifestyle]
tags: []
featured_image: 'images/demo/demo-portrait.jpg'
#scroll_image:
comments: true
share: true
---
```

When a new note is generated by creating a `[[link]]`, that note should come with come with some additional boilerplate besides the backlink - namely a Markdown header for a static site. Once that's squared away, it would be fantastic to have a lightweight editor that worked on mobile.  

Between these two additions, I think it would be a very fluid experience to seed, water, and tend your digital garden. Some additional workflows could be added to tag, group, and aggregate micro-blogs, but such is the life of a gardener. A solid website UI could make your blog just as useful to you as anyone else. What I mean by that is the end-user experience of _roaming_ on the blog should be as good or better than in the networked note-taking app itself.  

There's a few things out there that I'm keeping my eye on to pull this idea together:

- [Obsidian - integration with Github Pages](https://forum.obsidian.md/t/publish-to-web-e-g-via-github-pages/2060)
- [Obsidian - notes templates](https://forum.obsidian.md/t/templates-for-new-notes/322)  

---
# Farmer's Market  

Let's take this one step further and see if there are any commercial enterprise applications. In my current line of work, I can see an immediate use case for the ability to create a chain of bidirectional links. Here's a concrete example: 

1. Security Architecture defines requirements for a new service.
   - `[[Service Requirements]]`
2. DevOps team builds creates a set of tests according to the architecture. The wiki and/or README for the tests include a backlink to `[[Service Requirements]]`
   - `[[Service Requirements]]` now contains links to the codebase where the requirements are implemented.

This chain could extend laterally as well, aggregating different services and/or platforms. The ability to automatically create a summary of related artifacts without having to manage the links manually is certainly something that I personally would love in my working life.  

--- 

# Resources

- [Building a Second Brain](https://www.buildingasecondbrain.com/)
- [Zettelkasten](https://en.wikipedia.org/wiki/Zettelkasten)





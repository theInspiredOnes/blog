---
layout: post
title: Decoupling Code and Content with Containers
author: christoph
thumb: posts/decoupling/title.svg
excerpt_separator: <!--more-->
---
Most web developers who have worked on big sites or projects know this feeling: You have to change some part of the website and you are not sure whether changes in your CSS will break anything else on your site.<!--more--> You move around DOM elements and suddenly something breaks your design and now you have to go through hundreds of lines of CSS to find your selector and get everything right again. We all know that this shouldn’t happen and you should keep your code organized and clean, but we also know, it happens with reasonably sized projects eventually. Especially on big sites it is not an easy task to keep every bit of your code maintainable, flexible, and simple all the time. CSS might be easy to learn, but **writing good and scalable CSS is hard**.

In software development, we have tests. Tests make sure that everything crucial to the functionality of your software works. It gives you the confidence to refactor, because if you have written good tests, you can be certain the rest of your software still works as intended after you have made some changes to your code. We can’t test CSS. CSS needs *fiddling* and testing something that needs fiddling is [a waste of time](http://blog.8thlight.com/uncle-bob/2014/04/30/When-tdd-does-not-work.html).

If you have read [our last post](http://blog.the-inspired-ones.de/managing-kirby-plugins-with-git), you already know that we like to keep our code modular. It is also good style to keep parts of your software decoupled, so that they don’t have any undesired side-effects on different parts of your software and don’t rely on their surroundings. Those are coding practices we can use in our CSS, too!

## Concept of Containers

Back in November, we attended [beyond tellerrand](http://beyondtellerrand.com/berlin-2014/) in Berlin. The very last speaker was Oliver Reichenstein from the [Information Architects (iA)](https://ia.net). In his fantastic talk, he introduced a concept called *“Information Containers”* they used for the redesign of the Guardian. The basic concept allows you to design not full pages, but only parts of a page (containers) which can be combined to a full page. Sounds pretty much like a module in software development, but with a focus on content. To learn more about containers, you can read an [article on the Guardian](http://next.theguardian.com/blog/container-model-blended-content) about their new website.

The general definition of a container by iA is as follows:

* A container is a full width horizontal element which contains a collection of content.
* Pages hold a series of containers which are stacked on top of each other, the most important at the top and the least important at the bottom.
* Containers are self sufficient (hence the name) and as a result can easily appear in multiple locations in a variety of combinations.
* As the screen size gets smaller or larger, the content inside adjusts to the width of the screen.

Notice that containers are *self sufficient* and the content inside *adjusts to the width of the screen*. Turns out, [edenspiekermann_](http://www.edenspiekermann.com/) did [something similar](https://twitter.com/espiekermann/status/558871721671786496) for [ZEIT Magazin](http://www.zeit.de/zeit-magazin), but they call it *“Units“* (you should read the full conversation, it’s quite interesting). We don’t want to put a focus on information architecture here in this post, but want to show you how we can use the concept of containers or units to keep our code structured and maintainable. We can use the decomposition of a page for a decomposition of our code.

## Putting the Concept into Code

Assuming that we have designed our containers, we can use this to create *self sufficient modules* in our CSS. In the following we will give you a list of rules how you can implement such containers, units or whatever you would like to call it. If you are into [OOCSS](http://www.smashingmagazine.com/2011/12/12/an-introduction-to-object-oriented-css-oocss/), you are probably familiar with some of these principles.

### Self sufficiency

Containers should not rely on anything that is outside of their scope. They can be placed *anywhere* on the page, so your CSS should reflect that, too. As a result, make sure you don’t use selectors like `main h3` that can affect the appearance of objects in more than one container. In general, **avoid using the descendent selector**.

If you have self sufficient CSS, you can easily put it in its own partial (use the `@import` function of your preprocessor). You will have everything that belongs to one container in one place. Same thing goes for the HTML. Use Kirby’s [snippets](http://getkirby.com/docs/templates/snippets) or Jekyll’s [includes](http://jekyllrb.com/docs/templates/#includes) for your containers.

Each container is a [block](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/) in terms of BEM-style syntax. This means, every container-specific CSS-class will get its own prefix. **Don’t mix or `@extend` those container classes.** Create abstractions and use `@extend` or mixins to share common attributes.

Here’s an example. Let’s imagine we already have a `post` container which headline happens to be blue.

```html
<article class="post">
    <h1 class="post__headline">My Headline</h1>
    ...
</article>
```

```css
.post { ... }
.post__headline {
	color: blue;
}
```

Now you want to add a `comments` container which should have the same styling on the headline. The quick solution would be something like this:


```html
<section class="comments">
    <h1 class="post__headline">Comments</h1>
    ...
</section>
```

Or this:


```css
.comments { … }
.comments__headline {
	@extend .post__headline;
}
```

**Don't do this!** It will break your code and appearance if you change or remove the `post` container from your CSS. If you want both elements to share the same attributes, create an abstraction.

```css
.fancy-headline {...}

.post__headline {
    @extend .fancy-headline;
}

.comments__headline {
    @extend .fancy-headline;
}
```

You can also use regular OOCSS style extensions like `class="post__headline fancy-headline"`. You should also make sure that you [really want to use](http://csswizardry.com/2014/11/when-to-use-extend-when-to-use-a-mixin/) `@extend` and not a mixin. Everything that is unique to a container should be in its own CSS module. Create separate files for abstractions that can be shared between different containers. Don’t be afraid of long class names or having too many classes. More classes will help decoupling your code.

Since containers adapt to the the width of the screen, you should implement responsive behaviour right into your containers and place the code in the container-specific file. Only put container-specific code there. Use a grid system if you want to share responsive behaviour between containers.

If you now want to change something on your site, you only have to identify the container and look for its specific partial. You can also move around containers to different places across the DOM or even to a different page. You can *reuse* containers and easily add the same functionalities to different parts of your site.

## TL;DR;

If you create your site with a container-based architecture, you will be able to edit containers, put them anywhere on your site, or simply remove them without having to worry about possible side-effects on the rest of your site. Container-specific stylings stay in their own partial and have their own prefix, so you are able to quickly identify the selectors for elements you want to change on your page.

We hope you found this post helpful and that we could show you a way of keeping your CSS scalable and tidy. Use the comments if you have any questions or remarks. Let us know if you want to read more about advanced frontend techniques on this blog.

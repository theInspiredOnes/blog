---
layout: post
title: "Managing Kirby Plugins With Git"
author: martin
thumb: git-kirby-love.svg
---

Welcome to the first post on our new blog! In this post, we would like to show you how to use Git submodules, a great feature of the [Git version control system](http://git-scm.com/), to manage the different components of a Kirby website.

If you never worked with Git before, [try it](https://try.github.io/) and [learn it](http://www.git-tower.com/learn/ebook/command-line/introduction), first. It is a great tool everyone should know – writer, designer or developer.

## Kirby CMS

Kirby is a [powerful file-based CMS](http://getkirby.com/) written in PHP. It is easily hackable and extensible, and we are using it for almost all of our projects for the last several years. Over the time, we developed and published [quite](https://github.com/chbach/kirby-minigallery) [a](https://github.com/mzur/kirby-contact-form) [few](https://github.com/chbach/kirby-imgrd) [plugins](https://github.com/mzur/kirby-calendar-plugin) that made our developer life easier and working with Kirby even more fun. If you are into web design and development (you ended up on this blog post after all) and don't know it already, you definitely should give it a try.

A few months ago the highly anticipated version 2 of Kirby was released bringing lots of new features. Developers now could clone the new [Starterkit](https://github.com/getkirby/starterkit), always getting the latest releases of Kirby, the Kirby Toolkit and the Kirby Panel, each developed in their own Git repository. This setup of *automagically* syncing all the repositories made us curious since we never really put Git submodules into practice before.

<figure>
   <img src="/assets/images/starterkit-modules.png" width="200">
   <figcaption>Git submodules next to conventional directories of the Starterkit displayed on GitHub</figcaption>
</figure>

You can find a great [tutorial of how to use and manage the Starterkit with Git](http://getkirby.com/blog/working-with-git)—recursively cloning and updating the submodules—on the Kirby blog. In our blog post we like to go a step further and show you how to leverage the power of Git submodules for your own repositories and plugins.

## Git submodules

Git submodules offer the possibility to link Git repositories to other repositories, similar to a link in your local file system. You simply "hook up" a remote repository to a directory in your local repository. This empowers you to create setups like the Starterkit where you manage and develop different modules in different repositories, ultimately combining them as submodules. One advantage of this approach, besides decoupling the development of different modules, is the ability to easily update all your projects. You only have to run a single command and all your modules and plugins get updated to the latest release.

Working with Git submodules is no rocket science if you already are familiar with Git. Here is a small introduction on how to add, update and delete submodules.

### Adding

To add a submodule to your already existing Git repository you need the following command:

```bash
$ git submodule add <remote repository url> <submodule path>
```

That's it. The repository now will be cloned into the specified directory of the submodule path. If the added submodule itself contains other submodules you need a second command to initialize them, too:

```bash
$ git submodule update --init --recursive <submodule path>
```

You will notice a change of your `git status` after adding the submodule:

```bash
Changes to be committed:

   modified:   .gitmodules
   new file:   <submodule path>
```

The new directory of the submodule simply is a new file of your local repository that needs to be committed. More interesting is the `.gitmodules` file. It stores all the information of the submodules present in your local repository and gets updated whenever you add or remove submodules. Commit these changes and your new submodule is ready to go.

### Updating

To update a single submodule, go to the `<submodule path>` and run `git pull`. You can update all the submodules of your repository in a single batch command, too:

```bash
$ git submodule foreach --recursive git pull
```

Back in the root of your repository, you now will see the `git status`:

```bash
Changes not staged for commit:

   modified:   <submodule path> (new commits)
```

Add and commit these changes and the update of the submodule is complete.

### Deleting

Fully deleting a Git submodule is a little more complicated and requires multiple commands. First you need  to unregister the submodule with the `deinit` command:

```bash
$ git submodule deinit <submodule path>
```

This will clear the directoy of the submodule but won't remove it. So you need to remove it from the repository:

```bash
$ git rm <submodule path>
```

Now you'll see a new `git status`, which is the exact opposite of the one you got after adding a submodule:

```bash
Changes to be committed:

   modified:   .gitmodules
   deleted:    <submodule path>
```

Commit these changes and the submodule is removed.

But there is one thing left to do for a clean removal of the submodule. You still need to remove the internal Git files for the submodule:

```bash
$ rm -rf .git/modules/<submodule path>
```

If you omit this last step, you will run into problems if you want to re-add the removed submodule again.

Now we have all the tools for using our own submodules. But how can we apply this knowledge for an improved workflow with Kirby websites?

## Managing modules

If you are a developer, you probably already know that splitting up a project into small maintainable modules is usually a good idea. The same principle applies to a Kirby website, too. Kirby plugins are modules, or CSS frameworks, or virtually anything that you plug into your Kirby website that adds a bunch of new functionalities. All of these can be added as a Git submodule.

<figure>
   <img src="/assets/images/base-modules.png" width="350">
   <figcaption>Modules of one of our Kirby projects as Git submodules displayed in our GitLab</figcaption>
</figure>

But what to do with plugins that consist of multiple files like CSS, language files and Kirbytext extensions? A Git submodule is only a single directory after all. The answer is: links! Here is how we are currently managing our modules by the example of the [Contact Form plugin](https://github.com/mzur/kirby-contact-form).

### Directory structure

We always start with the Kirby Starterkit. We then add a new `modules` directory for all new Git submodules of the website (other than those of the Starterkit itself) to keep everything organized. So for adding the Contact Form plugin, we run:

```bash
$ git submodule add https://github.com/mzur/kirby-contact-form.git modules/kirby-contact-form
```

You will now end up with a directory structure similar to this:

```md
├── assets
├── content
├── kirby
├── modules
│   └── kirby-contact-form
│       ├── languages
│       ├── LICENSE
│       ├── README.md
│       ├── sendform
│       ├── sendform.css
│       └── snippets
├── panel
├── site
└── thumbs
```

### Linking

The Contact Form plugin requires copying several files to be installed. Having the plugin added as a Git submodule, we can't copy the files, otherwise all advantages of managing and updating the submodules would be lost. But we can *link* the files.

First, the `sendform` directory of the plugin should be placed to `site/plugins/`, so we create a link:

```bash
$ ln -s ../../modules/kirby-contact-form/sendform site/plugins/sendform
```

Second, `sendform.css` should be added to your CSS, so you also create a link:

```bash
$ ln -s ../../modules/kirby-contact-form/sendform.css assets/css/sendform.css
```

You get the idea. Note the use of *relative* links. If you create a link in `assets/css/`, you first need to go back two steps (`../../`) before you can go into the `modules/` directory containing the submodule. This enables us to use the same setup with the same links for different projects or on different machines.

The last step of installing the plugin is copying the language files. Here we can't simply link to the file of the submodule, since we might want to use multiple plugins each having their own language files. But we can create sort of a link from inside a PHP file, too. This is how the global language file of Kirby in `site/languages/en.php` can look like:

```php
<?php

$modulesRoot = kirby()->roots()->index . DS . 'modules';

include_once($modulesRoot . DS . 'kirby-contact-form' . DS . 'languages' . DS . 'en.php');

// add other plugins here

// overwrite plugin variables or add your own:
l::set('sendform-default-subject', 'Message from my super cool web form');
```

That's it. Like this, you can add and manage arbitrary modules, always keeping them up-to-date with a single command.

## Conclusion

In this post we have shown a way of adding, managing and updating Kirby components using Git submodules. A setup like this realls makes your life easier and gives you more time for the things you love most in creating a website: developing cool new features or designing a great look!

But you can go even further with Git submodules. While developing a new plugin, you can simply add the plugin repository as a submodule to a fully working development website. The plugin repository only contains the relevant files and commits of the plugin but you are able to develop and test it instantly in a living environment. You can even do instant hotfixes when you encounter bugs while working on a client project.

We hope we could show you how Git submodules are a valuable addition to your development workflow. In a future blog post, we might show you how to use multiple remote Git repositories to create and update whole client websites in the blink of an eye.

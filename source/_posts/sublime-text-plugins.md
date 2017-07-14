---
title: Sublime Text 3 的一些常用插件
categories:
  - Sublime Text
tags:
  - Sublime Text
permalink: sublime-text-plugins
date: 2017-07-07 08:45:42
description:
---
# Sublime Text 3 的一些常用插件

[TOC]

## Administrative
These plugins are kind of 'meta' because they are not concerned with writing code.

- Package Control
This package enables you to install other packages. Since build 3124, you can install it within Sublime via Tools ➡ Install Package Control.
- AdvancedNewFile
A better way to create new files. For instance, it automatically creates a folder when needed.
- SideBarEnhacements
Adds features such as renaming to the sidebar.
- A File Icon
Add icons to the files in the sidebar.
- ProjectManager
Organizing project files by putting them in a central location.
- TodoReview
Scans files for TODOs and more.
- FindKeyConflicts
Key conflicts are inevitably when you use a lot of plugins.
- Editor Config
Maintain consistent coding styles between different editors.
- Sync Settings
Keep settings in sync via Github-Gist.

<!-- more -->
## General
Useful for all languages.

- All Complete
Indexes all open files for auto-completion.
- BracketHighlighter
Improves the already built-in highlighting.
- Terminal
Open Terminal with current working directory set to the directory of the open file on a hot key.
- AlignTab
Align your code by `:`, `=`, `=>`, `%`, `,`,`|` or your own RegEx.
- GitGutter
Displays changes in the gutter (left to the line numbers).
- Git
Includes some git commands into Sublime.
- GitSavvy
Full git and GitHub integration.
- Gitignore
Fetches templates for the .gitignore provided by Github.
- DashDoc
Open current selection in Dash on a hot key.
- Local History
Keep a local history of your files. Even though I use git on almost every project, I still don't commit every change. It gives a better feeling to have the possibility to go back to every change.

## Javascript
- Tern for Sublime
Static Javascript code analyzer with auto-completion, function argument hints, 'go to definition' and more. The installation and configuration can be a little bit tricky but it's worth it. Choose Tern over SublimeCodeIntel (unmaintained) and JavaScript Completions (buggy).
- JavaScript & NodeJS Snippets
- Console Wrap
Fast way to log to console.
- JsPrettier
Integration of Prettier, the opinionated JavaScript formatter.
- Babel
Syntax definitions for ES6 JavaScript with React JSX extensions.
- TypeScript
- Elm Language Support

## HTML & CSS
- Sass
Sass is a preprocessor extending CSS and this plugins adds the language support.
- SassSolutions
Auto-complete variables/mixins from your 'settings.scss' file.
- CSS3
Replaces the built-in CSS support with a more up-to-date information. Includes cssnext support. Follow the instructions to reduce misbehaviour with other plugins.
- Emmet
Allows you to write HTML very fast. You have to learn their way though.
- Color Highlighter

## Linter
- SublimeLinter
- SublimeLinter-HTML-tidy
- SublimeLinter-contrib-stylelint
For CSS. Choose stylelint over SublimeLinter-CSSlint.
- SublimeLinter-contrib-SCSS-lint
- SublimeLinter-contrib-ESLint
- SublimeLinter-flow
- SublimeLinter-contrib-elm-make
- SublimeLinter-JSON

## Other
- omniMarkupPreviewer
使用 `ctrl+alt+o`打开浏览器实时预览

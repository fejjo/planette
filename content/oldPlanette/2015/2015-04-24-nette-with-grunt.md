---
date: "2014-04-24"
draft: false
title: "Nette with grunt"
tags: ["grunt", "javascript", "css", "minification"]
type: "blog"
slug: "nette-with-grunt"
author: "Honza Černý"
---

## Why this friendship?

Because everyone can have only **one javascript** and **one stylesheet** file. In era of mobile internet, when every request is expensive, we don't want header like this (in better example)

![](grunt-drupal.png)

What we want is that one minified javascript file and one minified file with stylesheets. I have got the best experience with Grunt and his great garden of plugins. I will show you simple example how to automate this task.

## What we need

Clean installation of Nette sandbox https://github.com/nette/sandbox and working Grunt http://gruntjs.com.

Grunt is JavaScript task runner. It’s similar to Make or Ant but written in JavaScript. Grunt has a lot of interesting plugins for better fronted development, but not only for frontend. Look at gruntjs.com/plugins.

How to install grunt-cli:

Grunt requires node.js in the latest version and a package manager npm http://nodejs.org.

When we already have npm with node.js installed, we can install grunt-cli to global space.
`npm install -g grunt-cli`

If everything works, after you run command `grunt --version` you should see something like this.

![](grunt-version.png)


Let's prepare working version of Nette Sandbox in ProjectGrunt folder. The best way how to do this, is by using composer.

`composer create-project nette/sandbox ProjectGrunt`

![](grunt-nette-sandbox-composer.png)


Check folder content.

![](grunt-folder.png)


After you configured web server and adjusted permission for log and temp folders (`chmod -R a+rw temp log`)  sandbox clone should work. If it doesn't work, you should follow [Quick start ](  http://doc.nette.org/en/2.1/quickstart/getting-started) guide.


## Configuration

Create file *package.json* in root folder. It contains list of node.js modules what we need to install (using npm).

```
{
  "name": "ProjectGrunt",
  "version": "1.0.0",
  "devDependencies": {
    "grunt": "~0.4.4",
    "grunt-usemin": "~2.1.0",
    "grunt-contrib-concat": "~0.3.0",
    "grunt-contrib-uglify": "~0.4.0",
    "grunt-contrib-cssmin": "~0.9.0",
    "grunt-nette-basepath": "~0.2.0"
  }
}
```

**grunt** - use latest version of Grunt

**grunt-usemin** - plugin for analysing our templates and prepare blocks with files for the next processes

**grunt-contrib-concat** - plugin for concatenation files

**grunt-contrib-uglify** - plugin for javascript minification

**grunt-contrib-cssmin** - plugin for css minification

**grunt-nette-basepath** - plugin for removing latte variable {$basePath} from file paths


Install these dependencies `npm install`

![](grunt-npm-install.png)

Create file *Gruntfile.coffee* in project root. You can use Gruntfile.js but I prefer Coffee for its expressiveness.

```
module.exports = (grunt) ->
  grunt.initConfig
    useminPrepare:
      html: ['app/templates/@layout.latte']
      options:
        dest: '.'

    netteBasePath:
      basePath: 'www'
      options:
        removeFromPath: ['app/templates/']

  # These plugins provide necessary tasks.
  grunt.loadNpmTasks 'grunt-contrib-concat'
  grunt.loadNpmTasks 'grunt-contrib-uglify'
  grunt.loadNpmTasks 'grunt-contrib-cssmin'
  grunt.loadNpmTasks 'grunt-usemin'
  grunt.loadNpmTasks 'grunt-nette-basepath'

  # Default task.
  grunt.registerTask 'default', [
    'useminPrepare'
    'netteBasePath'
    'concat'
    'uglify'
    'cssmin'
  ]
```

This includes definition of all tasks that will be loaded. And then it will be specified defaul main task which is run after command `grunt` and configuration for tasks useminPrepare and netteBasePath. For more information how plugin Usemin works visit https://github.com/yeoman/grunt-usemin.

In our case, we check config of netteBasePath task, where we said that basePath folder is in folder www, because Usemin plugin add to path folder, where file is located. We need to remove this path app/templates.

if you are running grunt under windows, this setting must be in windows style
```
removeFromPath: ['app\\templates\\']
```
(I know this bug, and I will fix it later)

## Run minification

Now we can run  `grunt` , but we got this error, because we don’t have blocks for usemin task in our template

![](grunt-usemin-error.png)

So we add these block to our template. In @layout.latte file we find classic javascript and css including and wrap it to build blocks.

```html
<!-- build:css {$basePath}/css/screen.min.css -->
<link rel="stylesheet" media="screen,projection,tv" href="{$basePath}/css/screen.css">
<!-- endbuild -->
<!-- build:css {$basePath}/css/print.min.css -->
<link rel="stylesheet" media="print" href="{$basePath}/css/print.css">
<!-- endbuild -->
```

and

```html
<!-- build:js {$basePath}/js/app.min.js -->
<script src="{$basePath}/js/jquery.js"></script>
<script src="{$basePath}/js/netteForms.js"></script>
<script src="{$basePath}/js/main.js"></script>
<!-- endbuild -->
```

and now  `grunt`  looks for block definitions and prepares files for next tasks.

![](grunt-success.png)

Great! Now we have minified files in folders, what is next?


## Version switching in Latte

For devel version we want uncompressed files, but for production we want use minified version.

How can we do this? I use configuration in neon.config where I define what version want to use and second config for version name to solve cache problems

*config.neon*

```yaml
#
# SECURITY WARNING: it is CRITICAL that this file & directory are NOT accessible directly via a web browser!
#
# If you don't protect this directory from direct web access, anybody will be able to see your passwords.
# http://nette.org/security-warning
#
parameters:
	site: # <---
		develMode: false # <---
		version: blackhawk # <---

php:
	date.timezone: Europe/Prague
	# zlib.output_compression: yes


nette:
	application:
		errorPresenter: Error
		mapping:
			*: App\*Module\Presenters\*Presenter

	session:
		expiration: 14 days


services:
	- App\Model\UserManager
	- App\RouterFactory
	router: @App\RouterFactory::createRouter
```

Update in BasePresenter so that templates know about these variables. Maybe it’s not the best solution, but good enough for this example.

*BasePresenter.php*

```php
/**
 * Base presenter for all application presenters.
 */
abstract class BasePresenter extends Nette\Application\UI\Presenter
{
	public function beforeRender()
	{
		parent::beforeRender();

		$this->template->production = !$this->context->parameters['site']['develMode'];
		$this->template->version = $this->context->parameters['site']['version'];
	}
}
```

and update *@layout.latte*

```html
{if $production}
	<link rel="stylesheet" media="print" href="{$basePath}/css/screen.min.css?{$version}">
	<link rel="stylesheet" media="print" href="{$basePath}/css/print.min.css?{$version}">
{else}
	<!-- build:css {$basePath}/css/screen.min.css -->
	<link rel="stylesheet" media="screen,projection,tv" href="{$basePath}/css/screen.css">
	<!-- endbuild -->
	<!-- build:css {$basePath}/css/print.min.css -->
	<link rel="stylesheet" media="print" href="{$basePath}/css/print.css">
	<!-- endbuild -->
{/if}
```

and

```html
{if $production}
	<script src="{$basePath}/js/app.min.js?{$version}"></script>
{else}
	<!-- build:js {$basePath}/js/app.min.js -->
	<script src="{$basePath}/js/jquery.js"></script>
	<script src="{$basePath}/js/netteForms.js"></script>
	<script src="{$basePath}/js/main.js"></script>
	<!-- endbuild -->
{/if}
```

and now our application uses minified version

![](grunt-final-sourcecode.png)

You can find whole example on github https://github.com/chemix/Nette-Grunt

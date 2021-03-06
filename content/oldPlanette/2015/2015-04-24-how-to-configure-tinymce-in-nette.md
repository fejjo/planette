---
date: "2014-04-24"
draft: false
title: "How to configure TinyMCE in nette"
tags: ["tinymce"]
type: "blog"
slug: "how-to-configure-tinymce-in-nette"
author: "Jak Trvdík"
---


**Usefull links**
- [Official site TinyMCE ]( http://www.tinymce.com/)
- [Configuration]( http://www.tinymce.com/wiki.php/Configuration)

## Attach TinyMCE to textarea

### By class (recommended)

```php
$form->addTextArea('text', 'Text')
	->setAttribute('class', 'mceEditor');
```

```js
tinyMCE.init({
	mode: "specific_textareas",
	editor_selector: "mceEditor",
	...
});
```

### By ID

```php
$form->addTextArea('text', 'Text')
	->setHtmlId('mceEditor');
```

```js
tinyMCE.init({
	mode: "exact",
	elements: "mceEditor",
	...
});
```


## Configuring validation

To enable validation in textarea with TinyMCe it is important to save written text to textarea before Nette validation.

If a form contains only one button (or if all buttons runs validation) you can attach text saving on `onSubmit` of the form.

```php
$form->getElementPrototype()->onsubmit('tinyMCE.triggerSave()');
```


```comment
Following is not tested, but hypothetically should work.
```

If one of the buttons doesn't run validation (i.e., "Back", set up by [setValidationScope ]( api:Nette\Forms\SubmitButton::setValidationScope())), validation is attached to `onClick` of those buttons, which run the validation.

```php
foreach ($form->getComponents(TRUE, 'SubmitButton') as $button) {
	if (!$button->getValidationScope()) continue;
	$button->getControlPrototype()->onclick('tinyMCE.triggerSave()');
}
```

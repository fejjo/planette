---
date: "2012-04-10"
draft: false
title: "Multiple use of single form"
tags: ["forms"]
type: "blog"
slug: "multiple-use-of-single-form"
---

Do you have form in separated class? Do you want to use it in several places?

<!--more-->

## Class form

```php
class LoginForm extends \Nette\Application\UI\Form
{
	/** @var \Nette\Security\User */
	private $user;

	/**
	 * @param \Nette\Security\User
	 */
	public function __construct(\Nette\Security\User $user)
	{
		parent::__construct();
		$this->user = $user;

		$this->addText('username', "Username");
		$this->addPassword('password', "Password");

		$this->addSubmit('sub', "Login");

		$this->onSuccess[] = [$this, 'process'];
	}

	public function process()
	{
		$values = $this->getValues();

		try {
			$this->user->login($values->username, $values->password);
		} catch (\Nette\Security\AuthenticationException $e) {
			$this->addError($e->getMessage());
		}
	}
}
```

Register callback processing the form during initialization.

## Use in presenter

```php
class LoginPresenter extends \Nette\Application\UI\Presenter
{
	/**
	 * @return LoginForm
	 */
	protected function createComponentLoginForm()
	{
		$form = new LoginForm($this->getUser());

		$presenter = $this;
		$form->onSuccess[] = function($form) use($presenter) {
			if ($form->valid) {
				$presenter->restoreRequest($presenter->backlink);
			}
		}
		return $form;
	}
}
```

Register second callback in factory, which will take action after valid processing of form.
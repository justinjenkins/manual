# Securing Applications

## Preventing SQL Injections

[SQL Injections](https://en.wikipedia.org/wiki/SQL_injection) are a common type of attack
and can take very different forms.

The ORM will quote any values that are passed as condition values automatically in order to 
protect your application from injection attacks.

```php
$author = 'UNTRUSTED USER INPUT';

Posts::find('first', array(
	'conditions' => array('author' => $author)
));
```

However you **cannot rely** on this feature if you use user input in any other place of a query. The 
following example shows how untrusted user input can be used safely used to specify the set of 
fields to be retrieved. As a countermeasure we first check if the provided field actually exists.


```php
$field = 'UNTRUSTED USER INPUT';

if (!Posts::hasField($field)) {
	throw new Exception('Invalid field.');
}

Posts::find('first', array(
	'fields' => array($field, 'title')
));
```

## Securing Form Fields and Values

The `FormSignature` class cryptographically signs web forms, to prevent adding or removing
fields, or modifying hidden (locked) fields.

Using the `Security` helper, `FormSignature` calculates a hash of all fields in a form, so that
when the form is submitted, the fields may be validated to ensure that none were added or
removed, and that fields designated as _locked_ have not had their values altered.

To enable form signing in a view, simply call `$this->security->sign()` before generating your
form. In the controller, you may then validate the request by passing `$this->request` to the
`check()` method.

Inside the view:
```
<?php $this->security->sign() ?>

<?= $this->form->create($post) ?>
	<?= $this->form->field('title') ?>
	<?= $this->form->field('body', array('type' => 'textarea')) ?>
	<?= $this->form->field('') ?>
	<?= $this->form->submit('save') ?>
<?= $this->form->end() ?>
```

Inside a controller action:
```php
if ($this->request->is('post') && !FormSignature::check($this->request)) {
	// The key didn't match, meaning the request has been tampered with.
}
```
	
<div class="note note-hint">
	To make form signing work, you must use the form helper to create the form and its fields.
</div>

When fields are inserted dynamically into the form (i.e. through JavaScript) then those can
be manually excluded while checking the signature.

```php
FormSignature::check($this->request, array(
	'exclude' => array(
		'_wysihtml5'	
	)			
));
```

## Preventing Mass Assignment

To prevent your application from opening up to the so called [mass assingment vulnerabilty](http://en.wikipedia.org/wiki/Mass_assignment_vulnerability), 
the framework provides you with the _whitelist_ feature. This whitelist can be used to limit the set of fields which get updated during create or update operations.

```php
$data = array(
	'name' => 'John Doe',
	'email' => 'haxx0r@example.org' // Added by the attacker.
);

$user = Users::findById($authed['user']['id']);
$user->save();
```

As a countermeasure additionally use the whitelist feature. So that in this
case really just the name field gets updated.

```php
$user = Users::findById($authed['user']['id']);
$user->save($data, array(
	'whitelist' => array('name')
));
```

<div class="note note-info">
	Always prefer whitelisting over blacklisting.
</div>

## Secure Your Database

[There has recently been a study](http://cispa.saarland/wp-content/uploads/2015/02/MongoDB_documentation.pdf) into how many people aren't properly securing their mongo databases. The study found that over 40,000 sites using MongoDB didn't correctly configure their databases to use a password so that a malicious user could connect without any verification. When putting your database system online, make sure that you properly configure your database so that it is secure. 

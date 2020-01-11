---
layout: post
title: AJAX Uniform
author: martin
thumb: posts/ajax-uniform/title.svg
---
**Heads up!** [Uniform v3](https://github.com/mzur/kirby-uniform) is there with lots of API changes so this tutorial is outdated. You can find a detailed example for an AJAX form over at [Read the Docs](https://kirby-uniform.readthedocs.io/en/latest/examples/ajax/).

Recently we got [asked](https://github.com/mzur/kirby-uniform/issues/55) how the [Kirby Uniform plugin](https://github.com/mzur/kirby-uniform) can be implemented using AJAX rather than the "old school" method by refreshing the page. While the latter is very reliable and robust (I hear there still are people around, having JavaScript disabled), some like it fancy. So let's see how we can make it work.

If you don't know how to use Uniform with Kirby yet, head over to our [previous post](/kirby-with-uniform) and come back later.

## The Concept

All [examples](https://github.com/mzur/kirby-uniform#examples) on the Uniform GitHub page show how a form can be implemented with the old school method, making use of the HTML form `action` attribute and sending the form data with a `POST` request to the same page containing the form. On the server, Kirby examines the form data and either performs the Uniform actions with it or returns the form page along with error messages.

Doing this with AJAX is not very different. Now you only need *two* Kirby pages, one displaying the form and one handling the AJAX requests. The page containing the form is plain HTML, flavoured with a little JavaScript to perform an asynchronous `POST` request instead of the default form `action`. The page handling the requests is basically just the controller of the old school form page, configuring Uniform and collecting the feedback information. Instead of returning HTML, the page handling the requests can simply return a JSON object, making it easy to parse and display the response with JavaScript on the form page.

So much for theory, now let's get our hands dirty.

## The Code

First we need to create the two pages, one containing the form and the other for handling the AJAX requests. Since they both belong together we can create the request handling page as a sub-page of the form like this:

```
content
├─ ...
└─ contact
   ├─ send
   │   └─ contact-send.en.txt
   └─ contact.en.txt
```

The `contact.en.txt` file can contain any additional information you want to display along with the contact form but in our case it only contains the title of the page.

`contact-send.en.txt` can be empty, too, but depending on the Uniform actions you like to perform you can add fields for convenient administration and configuration of the form. If you used the `email` action for example, you could add an `Email` field containing the email address of the receiver of the form data.

With our two pages created, we can go on implementing the controller.

### Controller

You got that right, we only need a controller for *one* page. Since the page containing the form doesn't do anything fancy, the template will suffice. Now go on and create the `site/controllers/contact-send.php` controller with the following content (we'll be using the `email` action as an example, of course):

```php
<?php

return function($site, $pages, $page) {

	$form = uniform(
		'contact-form',
		array(
			'required' => array(
				'name'      => '',
				'message'   => '',
				'_from'     => 'email'
			),
			'actions' => array(
				array(
					'_action' => 'email',
					'to'      => (string) $page->email(),
					'sender'  => 'info@example.com',
					'subject' => $site->title()->html() . ' - Message from the contact form',
				)
			)
		)
	);

	return compact('form');
};
```

Nothing special here; the controller looks just the same as the one you'd implement with the old school method.

### Templates

Let's start with implementing the template for the page containing the form. Create `site/templates/contact.php`, containing:

```html+php
<?php snippet('header') ?>

<form id="form" method="post">
	<label for="name">Name</label>
	<input type="text" name="name" id="name" required/>

	<label for="email">Email</label>
	<input type="email" name="_from" id="email" required/>

	<label for="message">Message</label>
	<textarea name="message" id="message" required></textarea>

	<label class="form__potty" for="website">
		Please leave this field blank
	</label>
	<input type="text" name="website" id="website" class="form__potty" />

	<input type="hidden" name="_submit" value="<?php echo uniform('contact-form')->token() ?>">

	<p id="feedback"></p>
	<button id="form-submit" type="submit">Submit</button>
</form>

<?php snippet('footer') ?>
```

If you are familiar with Uniform, you'll notice a few differences here. First of all there is almost no PHP involved. Besides the header and footer snippets only the form token is dynamically set as the value of a hidden form field.

We use this form field instead of the usual value of the submit button because of the way we'll send the form data later on. Be sure that you use the same Uniform ID (in this case `'contact-form'`) in the previously created controller and in this template, otherwise you'll get the wrong token and Uniform won't accept the requests.

Other than that we set a few IDs (`form`, `feedback` and `form-submit`) that make things a little easier with the JavaScript later on.

Let's now head over to `site/templates/contact-send.php`:

```php
<?php

header::contentType('application/json');
echo json_encode([
    'success' => $form->successful(),
    'message' => trim($form->message()),
    'errors' => array_keys(array_filter(get(), function ($field) use ($form) {
        return $form->hasError($field);
    }, ARRAY_FILTER_USE_KEY)),
]);
```
*This is the new version of the template making use of `json_encode` as suggested by [Robin Königsbrück](http://blog.the-inspired-ones.de/ajax-uniform#comment-2247132816). You can find the old version [here](https://gist.github.com/mzur/f3f1ff18ff8b4ad87fbb).*

The purpose of this template is to build a JSON object as a response to the AJAX requests we are about to send. This object contains information on whether the form was submitted successfully, the feedback message and an array of possible erroneous form fields.

We do this by using the `$form` variable passed on by the page controller just like we would do it the old fashioned way. Building JSON instead of HTML doesn't differ either, we only need to change the content type of the document using the `header` helper from the Kirby Toolkit.

Now we are ready to add the JavaScript to finally make everything work.

### JavaScript

For brevity, we'll use jQuery here but of course you can implement this in vanilla just the same.

For the JavaScript part, we'll need to modify the `contact.php` template once again. Add the following to the bottom of the template just above the footer snippet:

```html
<script type="text/javascript">
window.onload = function () {
	$('#form-submit').click(function (e) {
		e.preventDefault();
		$.post(
			'<?php echo $page->children()->find('send')->url()?>',
			$('#form').serialize()
		)
		.then(function (response) {
			var feedback = $('#feedback');
			feedback.removeClass('success error').text(response.message);
			$('input, textarea').removeClass('erroneous');
			if (response.success) {
				feedback.addClass('success');
				$('input, textarea').prop('value', '');
				$('#form-submit').prop('disabled', 'disabled');
			} else {
				feedback.addClass('error');
				for (var i = response.errors.length - 1; i >= 0; i--) {
					$('[name="' + response.errors[i] + '"]').addClass('erroneous');
				};
			}
		});
	});
};
</script>
```

Let's walk through that step by step:

```js
window.onload = function () {
	// ...
};
```

Since all the JavaScript of a page is usually loaded in the footer of a document, we'll need this to wait for jQuery to be loaded.

```js
$('#form-submit').click(function (e) {
	e.preventDefault();
	// ...
});
```

The form should be sent when clicking on the submit button, of course. Since the form has no `action` defined, nothing would happen otherwise. Just to be sure, we add a `preventDefault`.

```js
$.post(
	'<?php echo $page->children()->find('send')->url() ?>',
	$('#form').serialize()
)
.then(function (response) {
	// ...
});
```

This part actually sends the form data as a `POST` request. Since we added the JavaScript inline to the template, we can get the right URL to the request handling page by using Kirbys PHP methods. `$.post` returns a promise that gets resolved with the response JSON object built by our request handling template.

```js
var feedback = $('#feedback');
feedback.removeClass('success error').text(response.message);
$('input, textarea').removeClass('erroneous');
if (response.success) {
	feedback.addClass('success');
	$('input, textarea').prop('value', '');
	$('#form-submit').prop('disabled', 'disabled');
} else {
	feedback.addClass('error');
	for (var i = response.errors.length - 1; i >= 0; i--) {
		$('[name="' + response.errors[i] + '"]').addClass('erroneous');
	};
}
```

With the `response` object we can now display the feedback message and assign its class depending on whether the form was successfully sent or not. If it is not, the erroneous fields get an error class as well. Otherwise the form is cleared and the submit button disabled.

Note that we don't need to handle the old values of the form fields here. Since the page didn't refresh for the request, they just stay the same.

## Conclusion

That's it! With this setup you have a fancy AJAX form working with Uniform. Feel free to comment if you have questions or remarks. Contributors are very welcome on GitHub, too. We're currently (slowly) heading towards [v2.2](https://github.com/mzur/kirby-uniform/milestones/v2.2).

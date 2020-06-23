---
title: Be consistent while you're programming
author: ChienNV
date: 2020-06-24 12:00:00+07:00
tags: ['clean code', 'consistency']
---

# Trust is built with consistency

Consistency is one of the top standards for evaluating clean code. Being consistent means if you do something a certain way, do all similar things in the same way. It would help you reduce fragmentation, confusion during development.

So that the project ends up being more reliable, leads to rapid and smoother development.

# Why inconsistency is bad?

## Inconsistent code is harder to read and understand

Looking for an example from natural language. Let's compare two sentences:

- This is a normal sentence that everybody can understand
- tHiS iS A nOrmAl seNTenCE tHaT eVEryBOdy cAN uNDerStanD

You could read both sentences, but the first one is easier to read because it's consistent with how most people write English.

In programming, consistency in the code would reduce the time to read and understand code.

See this example below:

```python
active_products = [p for p in products if p.status == 'ACTIVE']

soldout_variants = filter(lambda v: v.status == 'SOLDOUT', variants)

active_suppliers = []
for supplier in suppliers:
	if supplier.status == 'ACTIVE':
		active_suppliers.append(supplier)
```

In all three paragraphs, I create a filtered list from another, but in three different ways.  You need to take more time to understand what am I doing, don't you?

I could make it more consistent by sticking with the first way, like:

```python
active_products = [p for p in products if p.status == 'ACTIVE']

soldout_variants = [v for v in variants if v.status == 'SOLDOUT']

active_suppliers = [s for s in suppliers if s.status == 'ACTIVE']
```

Now it's looking more understandable, right?

## Inconsistency could lead to fragmentation

In a frontend development, If APIs return different formats from a single source. We need to treat them differently.

See the example in two responses below:

 **POST products/ - Create a product**

```json
{
	"errors": "`title` has invalid characters; `cost` must be greater than 0"
}
```

**POST accounts/ - Create an account**

```json
{
	"errors": [
		"email": "invalid format",
		"age": "must be integer"
	]
}
```

For that case, you would need to maintain 02 functions to handle these messages:

```jsx
handleProductError(messages) {
}

handleAccountError(messages) {
}
```

So now, if we want to make changes, we have to duplicate them.

E.g Add log before errors happening

```jsx
handleProductError(messages) {
// add log here
}

handleAccountError(messages) {
// add log here, again
}
```

When you do the same things in different ways (and there's no reason to do it), that will make others need to handle those outputs in different ways. It could possible causes duplication or make the codebase more fragmental, more complex

## Inconsistency confuses new people

Now come up with a scenario, you're going to jump into a project. That project uses more than 10  technologies, 3 types of architectures to solve just one or two problems. What do you feel? What will you do?

It's a result of something called [Lava Layer](http://mikehadlow.blogspot.com/2014/12/the-lava-layer-anti-pattern.html). You may need to read all documentation, walk through all codebase to be able to understand this project.

Or in another case, a single term is written in various words in a project could be a trouble to someone who is learning it.

An inconsistent codebase is trouble maker to new people when they have to maintain it. That will be a problem when the team needs to scale up or gets support from outside.

# Some "best practices"

## Be consistent in naming

Things should have one and only one name through the project. It will be used in variable, comment, function, documentation,.. every place in your project.

If you intend to introduce a new term, make sure that it isn't existed before and everybody that related understand it. You may need to document it in a dictionary, that will help a lot.

## Be consistent in coding style

Pick a coding style and stick to it. There're many tools that help you keep the coding style consistent. Make sure that everyone in the team uses the same tool.

For Python, you can use [Black](https://github.com/psf/black) to format the code.

## Be consistent in APIs

Both for internal and public APIs.

For designing public APIs, make sure that your team uses same style, with URI, API methods, error messages, response's data....

For implementing internal APIs, reduce unstructure types when possible. Try to define an interface in your code. The freedom of script may bring the inconsistency.

## Be consistent in the way you work

Before starting, find an existed approach from the codebase and stick with it. All other approaches need to be reviewed carefully to balance the tradeoff between the benefits they bring and the inconsistencies they introduce.

Besides, You should refactor old stuff if you have a new approach which has better optimization. In case you don't have time, leave a comment that we have an inconsistency here, as to remind we need to fix it in the future.

---
title: My Worst API Design Mistakes
description: Some API design decisions are worse than others
date: 2022-05-23
layout: layouts/post.njk
---

Wise people learn from other's people mistakes, so here is a collection of bad API design decisions that ended in tears and regrets, ordered by worst offender first.
 
## Combining the first and last name in one field

We once designed an API where the customer's first & last name were combined in a single field as in:

```
{
	"customerName": "John Smith",
	...
}
``` 

At the time of the design, separating that value into two fields was deemed unnecessary, and providing a single input made the onboarding form look simpler, YAGNI so why make the data model more complex than necessary ?

Well it wasn't needed until it was; we had to integrate with another service that expected a first name & last name, and was quite strict about it. Surely it was possible to extract it from the existing data ? ... Right ? We looked at what we had in the database and it looked like this:

```
- John Smith
- Bob
- Smith Anna
- Mike Doe - Jane Doe
- Anna Mary Michell
- Industrytech LLC
- user-aefdefe
- ...
```

Around 15% of the data was complete garbage... It had to be fixed ...  So what's the way out ? 
- Add two new fields, with all the dance necessary to make them eventually mandatory and not break anything in the process
- Migrate the data you can migrate according to some hand crafted rules
- Adapt all onboarding process forms
- Adapt the frontend to ask the users that couldn't be migrated to update their data
- Some customers are not real persons, but companies ! Did we even support that business case ? What to do with them ?
- External communication & FAQ
- Support has to deal with the new questions, complains & migration mistakes
- What to do with customers that just don't update their data ? Drop them ? Flag them ? Fill data with placeholders ? 

All in all, a lot of work, just for a missing field. Point of the story; treat your customer information with care, structure & validate it well, it is more valuable than you think. And when you think that you aren't going to need something, also consider what you will need to do to adapt when you will.

## APIs with implicit parameters

Let's say you're making a book reading app, and want an API to let the user manage its list of books.

You design an api that looks like this

```
GET /api/v1/books
-> [
      { "id": 123, "title": "Moby Dick" }
   ]
```

This api returns the list of books, belonging to the user. You can also get books one by one, like this `GET /api/v1/books/123` , but it would only return the book if it belongs 



## Combining multiple resources into one api

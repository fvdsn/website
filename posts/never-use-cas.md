---
title: Never Ever Use Content Addressable Storage
description: Unless you really know what you are doing
date: 2025-10-07
layout: layouts/post.njk
---

Content Addressable Storage (often abbreviated as CAS) is a technique to
derive the id of a document from its content, basically by taking its hash.
Imagine you want to store pictures in a file store, and you need an id to reference
that picture, you simply take its hash.

This technique has benefits, first, if you have a document you automatically
know its id and can find other references to it. The other one is that two separate
processes are always going to use the same id for the document. But that is also its
biggest problem.

Let's say two users upload the same picture to your system. Since you're using CAS,
they're going to have the same id, and be essentially stored as one picture. Now
one of the users wants to delete the picture. Do you also delete it from your storage?
If you do, it's going to be deleted for the other user, and that's probably not what
you wanted.

So when a user wants to remove a CAS-addressed document, before really deleting it you
need to detect if it's the last reference. This is not easy to do, it is in fact much
harder to do correctly than eating the cost of storing duplicate files.

And usually when CAS is considered as a solution, it's to solve the need of deduplicating
files to save on storage. But even there, the good solution is to give files their own
internal uuids as storage keys, store its hash alongside, and generate external uuids for each file upload,
then use refcounts to handle the final delete.

So if you're considering CAS as a quick solution to that problem, I want to mention that
it is much easier to first store the files with uuids and no deduplication, then bolt on
deduplication afterwards than start with CAS and decide later on that it
was not needed.

There are of course a few good uses of CAS, such as git, where the whole point is that the
referenced version of a document is never deleted, or short term caches where it's always going to be
deleted.

That is at least my experience after working on systems that handled many many files.

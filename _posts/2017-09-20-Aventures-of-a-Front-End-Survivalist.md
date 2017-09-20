---
layout: post
title: "Adventures of a Front-End Survivalist"
date: 2017-09-20
---

*This post describes the challenges and solutions when trying to solve technical problems with minimal to no available infrastructure*

Got the requirement too late in the project phase to perform the standard solution design usually attributed to such projects.

Tom Hanks had more kit on Castaway than the scraps I had available to build my solution, yet I pushed on determined to squeeze potentially the last drop of value from the aging tools at my disposal.

The requirement was to push data regularly to a remote API endpoint, either automatically as JSON or as an XML file upload.

My daydream of using PySpark to grab the data automatically from source systems using their APIs or via ODBC connections was cut short when consdering that our IT team is already stretched and under-resourced and the new requirement was almost certainly not on their radar for the immediate future.

As I resigned myself to the fact that any short-term solution capable of meeting the pending deadline would likely involve manual intervention, I would have to build something as simple as possible...something like a single page web application with no more than a few options. Fool proof-ish.

But how would I make it happen? And with what tools? Is is even possible without proper infrastructure to implement such a solution? If it is, then certainly it would not be recommended for a long term solution. Nevertheless, as the British say, keep calm and carry on.

I knew from my front-end experience that jQuery, d3, and even base JavaScript all had capabilities that allowed for XML/JSON manipulation, so my first challenge was to convert a dataset into the target dataset specification as supplied by the organisation managing the remote endpoint.

The ever helpful Google points me in the direction of an old post on the O'Reilly website which describes how to create nested XML documents (strings) quickly with just a couple of functions. That would be helpful later, so I gathered this nugget into my tech basket and continued.



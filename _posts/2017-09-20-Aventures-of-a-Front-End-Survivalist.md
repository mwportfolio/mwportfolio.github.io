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

The ever helpful Google points me in the direction of an old post on an [O'Reilly website](http://archive.oreilly.com/pub/h/2127) which describes how to create nested XML documents (strings) quickly with just a couple of functions. That would be helpful later, so I gathered this nugget into my tech basket and continued.

~~~ javascript
// Bare bones XML writer - no attributes
function element(name,content){
    var xml
    if (!content){
        xml='<' + name + '/>'
    }
    else {
        xml='<'+ name + '>' + content + '</' + name + '>'
    }
    return xml
}
~~~

So far, so good. But the end-user for whom this solution is designed is unlikely to care about such tech efficiencies. Rather they are likely to want a simple interface where they can load a source file, and a capability to retrieve the automagically-converted XML file.

The next challenge therefore was loading a file into the webpage. I knew this was possible with HTML/JavaScript but perhaps there is a more modern, elegant solution. 

HTML5 has a capability called FileReader which handles reading files into memory and making them accessible to the web page.

The source dataset that my user will likely be uploading will be structured. Ideally it will contain a consistent structure so that my web app can consistently produce the desired XML output.

The CSV format is generally available as an export option through most data/BI tools and as a bonus can be read and manipulated using Excel. Furthermore there is a library called [PapaParse](http://papaparse.com/) which handles parsing CSV files which will surely come in handy for my solution, so into the tech basket it goes.

The following snippet opens the "File Open" dialog where the user can select a file accessible to them to be imported into memory and processed.

~~~ javascript
var file = this.files[0];
var reader = new FileReader();
reader.onload = function(progressEvent) {
    Papa.parse(file, {   
        header: true,   
        complete: function(results) {    
            parsed_csv = results.data;    
            processResults(results);   
        }  
    }); 
}  

reader.readAsText(file);
~~~

So now, all within a browser, I have capability to:
- load and process a structured CSV file, and 
- efficiently manufacture a XML string.

The final structural challenge was to provide the XML string as a "downloadable" or saveable XML file, which could then be uploaded to the remote XML endpoint.

The web has come a long way. God bless it and its inhabitants. Some time ago, Eli Grey developed the [FileSaver.js](https://eligrey.com/blog/saving-generated-files-on-the-client-side/) JavaScript library which among other things allows the saving of documents from strings, all within the browser.

The final piece in my basket of tech nuggets. Now I could offer the user an XML file to download/save, from the XML string I manufactured, from the CSV file uploaded by the user.

Once the library was loaded, a couple of lines provided the final functionality:

~~~ javascript
var blob = new Blob([xmlString], {type: "text/xml"});
saveAs(blob, "test.xml");
~~~

All libraries used were accessed via CDN servers, making the app quite portable with no dependencies, only a single HTML file with all functionality baked in.

Adding Bootstrap CSS for styling the app gave it a modern twist that will hopefully makle it feel more familiar and palatable to the end user.

![Screenshot](https://github.com/mwportfolio/mwportfolio.github.io/raw/master/screenshots/CSV-to-XML-Conversion-Screenshot.png)

So even without any servers, infrastructure, automated scheduling etc there is still plenty of features and capabilities to squeeze out of even some of the oldest tech laying about the place.

Thanks for reading about my survival adventures in front-end development.

---
layout: post
title:  "Werkzeug Debugger"
description: "One of the most popular WSGI utility frameworks for Python is Werkzeug. It simplifies the handling of HTTP connections within your Python application, however if left exposed to the public internet, could have disaterous consequences for website owners."
date:   2018-10-03 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
One of the most popular WSGI utility frameworks for Python is Werkzeug. It simplifies the handling of HTTP connections within your Python application but also provides a powerful debugger that permits one to execute code from within the browser.

![](/assets/werkzeug_logo.png "Werkzeug Logo")

While the provided link warns those to not enable the debugger on anything production, it is often ignored or forgotten about and ends up being enabled in the first place. It is possible to search for systems on the Internet that have the debugger enabled and execute Python code remotely.

It should be noted that this affects both the Flask and Django frameworks mainly but could be used elsewhere. Revealing and using the debugger Getting the debugger to reveal itself is fairly simple: cause an exception. This can be achieved by writing some faulty code or by simply making it happen yourself such as in this code example using the Flask framework:
<pre>
from flask import Flask

app = Flask(__name__)

@app.route('/')
def main():
    raise

app.run(debug=True)
</pre>
All you need to do is execute the above as a Python script and the following should be outputted:
<pre>
$ python flasktest.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader...
</pre>
Now view the page via the URL it provides and you'll have access to the debugger as follows:

![](/assets/Werkzeug_Debugger_1.png)

This is basically Remote Code Execution by design.

An RCE is basically game over. You can inject code directly to the application, exposing all data on the server which the application has access to.

Both the documentation of Werkzeug and Flask mentions this with large bold letters that you should [not expose](http://werkzeug.pocoo.org/docs/0.10/debug/#enabling-the-debugger) this [debugger online](http://flask.pocoo.org/docs/0.10/quickstart/#debug-mode).

Now, Werkzeug requires an actual error to trigger the console, as it uses a secret key generated when the application starts, which is only exposed in the Werkzeug Debugger page. Without this secret key you cannot run any commands, that’s why you need an exception to reveal the secret. Also worth noting is that the debugger only [accepts commands sent in by the GET-parameter](https://github.com/mitsuhiko/werkzeug/blob/d4e8b3f46c51e7374388791282e66323f64b3068/werkzeug/debug/__init__.py#L169), which will then show up in access logs on the vulnerable host, which is great for forensic analysis and investigation.

Also, each line in the code responds to a frame which is also needed for the debugger to know exactly where in the code to run the command.

A request is then made to:
 * [http://example.com/?__debugger__=yes&cmd=print+%221%22&frm=[FRAME]&s=[SECRET]](http://example.com/?__debugger__=yes&cmd=print+%221%22&frm=[FRAME]&s=[SECRET])
which will return the result of the command.

**Important: Werkzeug 0.11+ require knowledge of a PIN, after several unsuccessful attempts the debugger will shutdown**

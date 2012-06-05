---
date: '2010-07-01 08:00:37'
layout: post
slug: pythons-tornado-has-swept-me-off-my-feet
status: publish
title: Python's Tornado has swept me off my feet
wordpress_id: '457'
categories:
- Development
- Infrastructure
- Python
tags:
- API
- asynchronous
- non-blocking
- Python
- REST
- tornado
- web.py
---

I've been working with Python's [Tornado](http://www.tornadoweb.org/) for about 2 months now and I love it.

Tornado is a non-blocking web server written in Python.  It's structure is similar to web.py so users of that popular Python web framework will feel right at home.  This is a structure that lends itself really well to developing RESTful APIs as the methods you write to handle incoming requests are named after the HTTP methods used:

```python
class PlaceHandler(tornado.web.RequestHandler):
    def get(self, id):
        # respond to a GET
        self.write('GETting something')

    def post(self):
        # respond to a POST
        self.write('POSTing something')
```

You match URI paths to "handlers" (the _controller_ for those MVC folk) via a list of regex, handler tuples that instantiate an "application".

```python
application = tornado.web.Application([
    (r"/place", PlaceHandler),
    (r"/place/([0-9]+)", PlaceHandler)
])

if __name__ == "__main__":
    http_server = tornado.httpserver.HTTPServer(application)
    http_server.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

As usual any values that are captured from the regex are passed, in order, to the method that receives the request in the handler.

Because of it's non-blocking nature Tornado bundles an asynchronous HTTP client for use internally.  Additional modules include a command line and config file convenience library, escaping, 3rd party authentication (Facebook, Twitter, etc.), a wrapper around MySQLdb, and templating.  All in all this makes it a formidable web framework in its own right, especially if you're looking for something that's light and [FAST](http://www.tornadoweb.org/documentation#performance).

In production, I'm running 4 Tornado instances per server behind [nginx](http://nginx.org/).

One issue not addressed out of the box was daemonizing the Tornado instance.  I added PID file management and the ability to daemonize as follows (pid.py module follows):

```python
# capture stdout/err in logfile
log_file = 'tornado.%s.log' % options.port
log = open(os.path.join(settings.log_path, log_file), 'a+')

# check pidfile
pidfile_path = settings.PIDFILE_PATH % options.port
pid.check(pidfile_path)

# daemonize
daemon_context = daemon.DaemonContext(stdout=log, stderr=log, working_directory='.')
with daemon_context:
    # write the pidfile
    pid.write(pidfile_path)
    
    # initialize the application
    http_server = tornado.httpserver.HTTPServer(application.app)
    http_server.listen(options.port, '127.0.0.1')
    
    try:
        # enter the Tornado IO loop
        tornado.ioloop.IOLoop.instance().start()
    finally:
        # ensure we remove the pidfile
        pid.remove(pidfile_path)
```

And now the pid.py module:

```python
# pid.py - module to help manage PID files
import os
import logging
import fcntl
import errno


def check(path):
    # try to read the pid from the pidfile
    try:
        logging.info("Checking pidfile '%s'", path)
        pid = int(open(path).read().strip())
    except IOError, (code, text):
        pid = None
        # re-raise if the error wasn't "No such file or directory"
        if code != errno.ENOENT:
            raise
    
    # try to kill the process
    try:
        if pid is not None:
            logging.info("Killing PID %s", pid)
            os.kill(pid, 9)
    except OSError, (code, text):
        # re-raise if the error wasn't "No such process"
        if code != errno.ESRCH:
            raise

def write(path):
    try:
        pid = os.getpid()
        pidfile = open(path, 'wb')
        # get a non-blocking exclusive lock
        fcntl.flock(pidfile.fileno(), fcntl.LOCK_EX | fcntl.LOCK_NB)
        # clear out the file
        pidfile.seek(0)
        pidfile.truncate(0)
        # write the pid
        pidfile.write(str(pid))
        logging.info("Writing PID %s to '%s'", pid, path)
    except:
        raise
    finally:
        try:
            pidfile.close()
        except:
            pass

def remove(path):
    try:
        # make sure we delete our pidfile
        logging.info("Removing pidfile '%s'", path)
        os.unlink(path)
    except:
        pass
```

I'm going to follow up this post another on how I added a simple concept of "models" and an easy way to perform MySQL transactions.  Let me know if you have any specific questions!

LogTrace
========

[![Build Status](https://travis-ci.org/paul-wolf/logtrace.svg?branch=master)](https://travis-ci.org/paul-wolf/logtrace)

Aggregate messages to produce a log entry representing a single event or procedure.

```
import logging
from logtrace import LogTrace

logger = logging.getLogger(__name__)
trace = LogTrace(logger=logger)

trace.add("Let's get started")
...
trace.add("Later, something else happens")
...
trace.add("And finally...")

trace.emit()
```

You get a single log entry like this:

```
[05/Jan/2018 11:11:00] DEBUG [21, .30s] Let's get started; [65, .132s] Later, something else happens; [75, .330s] And finally...
```

The purpose of this module is to easily asssociate log messages
together that belong together.

Install
-------

	pip install logtrace

Note that this only suppports Python 3. Let me know if anyone wants support for Python 2. There are no dependencies outside the Python Standard Library for this module. 

We respect logging levels. So, the overhead of using LogTrace is minimal if your log level is not effective. If your log level is `logging.INFO` and you call `logtrace.emit_debug()`, almost all overhead is avoided minus some function call overhead and one or two conditional expressions. 

What LogTrace is *not*: This is *not* a logging framework. LogTrace uses the standard Python `logging` module. All your configuration to `logging` is going to be used by LogTrace. All your handlers are going to act exactly as before. If you use a framework like Django, you use it just like you do now. No changes whatever are required to your logging configuration. 

We also provide other features like

* Easily generate a UUID for the logged event.

* Timings for each message.

* Frame information for each part message, like filename, function, lineno

* Any logging mechanism can be used, not just standard Python logging.

* Pass structured data (json).

* Enable log messages to be parsed

We wanted to provide something that works in perfect harmony with the
existing Python logging module without unnecessary duplication of
features and no external dependencies (outside the PSL).

```
    LogTrace(logger=None,      # we'll emit output here
             delimiter="; ",   # delimiter between messages
             tag='',           # add a non-unique label 
             unique_id=False,  # create a uuid to identify the log?
	     verbosity='v'     # level of output for frame information
            )
```

* `logger`: the standard logger returned from `import logging; logger
  = logging.getLogger(__name__)`. You can create a `LogTrace()`
  without a logger in which case it creates one called `__name__`.

* `delimiter`: the character(s) used between messages

* `tag`: This is a convenience to tell LogTrace() to use hash+tag at
  the start of every entry after calling `.emit()` for ease of
  searching.

* `unique_id`: generate a uuid to associate with the final message output.

* `verbosity`: v, vv, vvv for three levels of verbosity when adding
  frame information

`LogTrace.get_uid()`: return the unique id. If one has not be set during construction of the LogTrace, a uuid is generated. Otherwise, it returns the existing one. 

`LogTrace.set_uid(uid)`: Set a unique id. This can be done by constructing `LogTrace()` with `unique_id=True`. This takes normally either a uuid or str argument.  

`LogTrace.add(msg, data, backup)`: Add a message to the list. This will get frame information for the call depending on the verbosity level. 

`LogTrace.emit_string()`: return a string that is the final log message

LogTrace.emit(): call `logger.debug(message)` 

LogTrace.emit_error): call `logger.error(message)`

LogTrace.emit_info(): call `logger.info(message)`

LogTrace.emit_debug(): call `logger.debug(message)`

LogTrace.emit_warning(): call `logger.warning(message)`

LogTrace.emit_critical(): call `logger.critical(message)`

When the `LogTrace` is created, `time.time()` is recorded. Whenever `LogTrace.add()` is called, the time is recorded for when the message is added. The final message prints the number of seconds since creating.

You probably want to avoid including `LogTrace.add()` in loops. You also probably want to create it as a local, not a module-level variable. Pass it as a method argument rather than using a module level instance. If you do want to re-use a `LogTrace` and clear messages, you can call `LogTrace.clear()`. But be aware the uid might need to be reset depending on your application requirements. 

Testing
-------

	pip install pytest
	cd logtrace
 	pytest test.py --verbose

or

	python logtrace/test.py



Performance
-----------

`LogTrace()` appends to a list of strings everytime you call `add()`. But it firstly calls `insepct.getFrameInfo()` and builds the string with that information. When `emit()` is called, it concatenates all the strings in the list separated by `delimiter` and then calls `logger.info()` or whatever method is appropriate. If the effective level is not the current level for the method, then the list will be empty and it won't do the call to the `logger` method.

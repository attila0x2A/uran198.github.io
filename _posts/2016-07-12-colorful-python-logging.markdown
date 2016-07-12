---
layout: post
title: Colorful Python logging
comments: true
categories: en python
---

Some developers prefer inserting random prints in their code to debug it. This approach worked for me too a lot of
times. But when the time comes to writing big project it might not be the best one. First of all, it's quite some work
to add and then delete all the prints. You have no distinction between important and not important prints. Also, if
something will go wrong, for example on server, it may take some time to reproduce the bug with prints while you, or
someone else, spotted it without them.

Because of this, it's better to use logging. Python comes with very good logging module. It is very powerful and lets
you customize all sorts of things. With it you can create custom handlers, or use existing ones (there is one for almost
all your needs), you can tweak the format of the messages shown, create new log levels. You can even create your own
handlers and formatters based on existing ones or from scratch.

Also, different loggers can be connected with each other with parent-child relationship. This way you can configure
parent and it will propagate all the settings to it's children, which makes code really DRY.

Don't forget that logs are thread safe, so you won't have to worry about that, either.

About all of those things you can read in the [documentation](https://docs.python.org/3/library/logging.html). It is
extremely well written and you can start with loggers in the matter of minutes.

There is one feature though, I missed a lot from standard logging. I like colours in my terminal and by default python's
logging don't produce coloured output (which makes a lot of sense, actually). But this can easily be solved with making
custom logger formatter.

First of all, what gives colours in terminal? It turns out there are special escape sequences for ANSI/VT100 terminals
and terminal emulators (which I think are supported in most of the modern terminals as well). With them you can change
not only colour, but also brightness, font and some other attributes.

So for example to make something red, you can write:

```python
print("\x1b[31mRed part\x1b[0m Regular part")
```

The "Red part" will be displayed in red.

Here basically, you're using special escape sequence, `\x1b` stands for `<Esc>`, `\x1b[31m` is for the red foreground
and `\x1b[0m` - to reset all the attributes. You can take a look at other sequences
[here](http://www.termsys.demon.co.uk/vtansi.htm).

I don't like memorizing different sequences and they may look a little awkward in the code and they may be platform
dependent. Apart from that, I'm quite a lazy person, so I'll use [`colorama`] to produce this sequences for me.

What I want is to display the severity of the log in respective colour, e.g. errors with red, and warnings with yellow.
I don't know the severity of the log record, before it's created, so I'll need my own custom formatter. Otherwise, I could go
away with just specifying colours in the format string.

Let's dive into code:

```python
import logging
import colorama
import copy

# specify colors for different logging levels
LOG_COLORS = {
    logging.ERROR: colorama.Fore.RED,
    logging.WARNING: colorama.Fore.YELLOW
}


class ColorFormatter(logging.Formatter):
    def format(self, record, *args, **kwargs):
        # if the corresponding logger has children, they may receive modified
        # record, so we want to keep it intact
        new_record = copy.copy(record)
        if new_record.levelno in LOG_COLORS:
            # we want levelname to be in different color, so let's modify it
            new_record.levelname = "{color_begin}{level}{color_end}".format(
                level=new_record.levelname,
                color_begin=LOG_COLORS[new_record.levelno],
                color_end=colorama.Style.RESET_ALL,
            )
        # now we can let standart formatting take care of the rest
        return super(ColorFormatter, self).format(new_record, *args, **kwargs)
```

And that's it. Now to use it, we can do the following:

```python
# we want to display only levelname and message
formatter = ColorFormatter("%(levelname)s %(message)s")

# this handler will write to sys.stderr by default
handler = logging.StreamHandler()
handler.setFormatter(formatter)

# adding handler to our logger
logger = logging.getLogger(__name__)
logger.addHandler(handler)

logger.error("This is an error!")
logger.warning("This is a warning!")
```

This is basically it.

If you have any questions, suggestions or correction, please feel free to leave them in the comments.

---
layout: post
title: Colorful Python logging
categories: en python
---

Some developers prefer inserting random prints in their code to debug it. This approach worked for me too a lot of time.
But when the time comes to writing big project it might not be the best one.
First of all, it's quite some work to add and then delete all the prints afterwards. You have no distingshion between
imprtant and not important prints. Also, if something will go wrong for example on server it may take some time to
reproduce the bug with prints while you or someone else spotted them without them.

Because of this it's better to use logging. Python come with very good logging module. It is very powerfull and let's
you customize all sorts of things. With it you can create custom handlers, or use existnig ones (there is one for almost
all your needs), you can tweak the format of the messages shown, create new log levels. You can even create your own
handlars and formatters based on existing ones or from scratch.

Also, different loggers can be connected with each other with parent-child relationship. This way you can configure
parent and it will propagate all the settings to it's children, which makes code really DRY.

Don't forget that logs are thread safe, you won't have to worry about that, either. 

About all of those things you can read in the [documentation](). It is extremely well written and you can start with
loggers in the matter of minutes.

There is one feature though, I missed a lot from standart logging. I like colours in my terminal and by default python's
loging don't produce coloured output (which makes a lot of sence, actually). But this can easily be solved with making
custom logger formatter.

First of all, what gives colours in terminal. It turns out there are special escape sequances for ANSI/VT100 terminals
and terminal emulators (which I think are supported in most of the modern terminals as well). With them you can change
not only colour, but also brigthness, font and I think some other attributes. 

So for example to make something red, you will write:

```python
print("\x1b[31mRed part\x1b[0m Regular part")
```

The "Red part" will be displayed in red.

Here basically, you're using special escape sequence, `\x1b` stands for `<Esc>`, `\x1b[31m` is for red foreground and
`\x1b[0m` to reset all the attributes. You can take a look at other sequences [here](http://www.termsys.demon.co.uk/vtansi.htm).

I don't like memorizing different sequenes and they may look a little aquard in the code. Also, I'm quite a lazy person,
so I'll use [`colorama`] to produce this sequences for me.

What I want is to display the severity of the log in respective colour, e.g. erros with red, and warnings with yellow.
I don't know ahead of time of severity log record will be, because of this I need my own handler. Otherwise, I could go
away with just specifying colours in the format string.

Let's dive into code:

```python
import logging
import colorama

class ColorFormatter(logging.Formatter):
    def format(self, record, *args, **kwargs):
        if record.levelno in LOG_COLORS:
            # we want levelname to be in different colour, so let's modify it
            record.levelname = "{colour_begin_seq}{levelname}{colour_end_seq}".format(
                levelname=record.levelname,
                colour_begin_seq=LOG_COLORS[record.levelno],
                colour_end_seq=colorama.Style.RESET_ALL,
            )
        # now we can let standart formatting take care of the rest
        return super(ColorFormatter, self).format(record, *args, **kwargs)
```

And that's it. Now to use it, we can do the following:

```python
# we want to display only levelname and message
formatter = ColourFormatter("%(levelname)s %(message)s")

# this handler will write to sys.stderr by default
handler = logging.StreamHandler()
handler.setFormatter(ColorFormatter)

# adding handler to out logger
logger = logging.getLogger(__name__)
logger.addHandler(handler)
```

This is basically it.

If you have any questions, suggestions or correction, please feel free to leave them in the comments.

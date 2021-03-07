---
title: Logging in Python
tags:
  - gatsby
  - code-notes
emoji:
---

## Basic example

In the following example, we specify two handlers. So the messages will output to `stdout` and to the `log_file` at the same time.

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

formatter = logging.Formatter(
    "%(levelname).1s%(asctime)s [%(module)s:%(lineno)d] %(message)s",
    datefmt="%m%d %H%M%S",
)

stream_handler = logging.StreamHandler(sys.stdout)
stream_handler.setFormatter(formatter)
file_handler = logging.FileHandler(log_file, mode="a")
file_handler.setFormatter(formatter)

logger.handlers = [stream_handler, file_handler]

logging.debug("this is debug")
logging.info("this is info")
logging.warning("this is warning")
logging.error("this is error")
logging.critical("this is critical")
```

Example output:

```
I0306 205253 [<ipython-input-4-3747c4192e4e>:17] this is info
W0306 205253 [<ipython-input-4-3747c4192e4e>:18] this is warning
E0306 205253 [<ipython-input-4-3747c4192e4e>:19] this is error
C0306 205253 [<ipython-input-4-3747c4192e4e>:20] this is critical
```

## Colorful example

It has dependency on a non-builtin package, `colorma`. Also it will be more difficult to tweak the settings. However, colorful messages could be useful in Jupyter.

```python
import logging
from colorama import Fore, Back, Style


class ColoredFormatter(logging.Formatter):
    """Colored log formatter."""

    def __init__(
        self, *args, colors: Optional[Dict[str, str]] = None, **kwargs
    ) -> None:
        """Initialize the formatter with specified format strings."""

        super().__init__(*args, **kwargs)

        self.colors = colors if colors else {}

    def format(self, record) -> str:
        """Format the specified record as text."""

        record.color = self.colors.get(record.levelname, "")
        record.reset = Style.RESET_ALL

        return super().format(record)


formatter = ColoredFormatter(
    "[{asctime} {name}] {color}{message}{reset}",
    style="{",
    datefmt="%m%d %H:%M:%S",
    colors={
        "DEBUG": Fore.LIGHTBLACK_EX,
        "INFO": Fore.GREEN + Style.BRIGHT,
        "WARNING": Fore.BLACK + Back.YELLOW + Style.BRIGHT,
        "ERROR": Fore.WHITE + Back.LIGHTRED_EX,
        "CRITICAL": Fore.WHITE + Back.LIGHTMAGENTA_EX,
    },
)

handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(formatter)
```


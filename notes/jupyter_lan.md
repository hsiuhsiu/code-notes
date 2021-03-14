---
title: My Jupyter Setep Note
tags:
  - jupyter
  - env
emoji:
---

## LAN Access

> **_NOTE:_**
>
> Google the instructions for details steps

- Use `Jupyter notebook password` to set a password. It will write the hash of the password to `jupyter_notebook_config.json` in `~/.jupyter`. We will use that as our default config.
- More settings added to the json config:
	- 	```json
    "notebook_dir": "/Users/<username>/my/path",
    "ip": "xxx.lan"
    ```
- Now it can be accessed by xxx.lan:8888 from LAN

## Formatter (in Jupyter Lab)

- Follow the instruction in this [doc](https://jupyterlab-code-formatter.readthedocs.io/en/latest/how-to-use.html)
  - Unmentioned dependency: node
  - Possible hiccup:
    - Missed module: install it
    - Version mismatch:, check the FAQ
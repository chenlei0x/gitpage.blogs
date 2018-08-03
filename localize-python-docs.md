---
title: localize python docs
date: 2018-08-03 20:39:49
tags: [python,doc]
categories: python
---

# Localize python docs
```bash
# in case of messing up your dir, make a new one
mkdir test && cd test
# download html format docs to local
wget https://docs.python.org/3/archives/python-3.7.0-docs-html.tar.bz2
# extract the tar package
tar xvf python-3.7.0-docs-html.tar.bz2
# the step above will extract the content to python-3.7.0-docs-html
cd python-3.7.0-docs-html
# start a httpserver with python3
python3 -m http.server 8000
# access 127.0.0.1:8000 with your browser
chrome http://127.0.0.1:8000
```

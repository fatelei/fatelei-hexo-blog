title: Using dozer to debug memory leak with gunicorn
date: 2016-04-03 10:14:14
tags: [dozer]
---

+ app.py

```
# -*- coding: utf8 -*-

import subprocess

from dozer import Dozer
from flask import Flask


app = Flask(__name__)


@app.route("/")
def home():
    subprocess.call(["ls", "-l"])
    return "hello world"

app = Dozer(app)
```

```
$ gunicorn -b 0.0.0.0:5000 app:app
```

+ 打开浏览器

访问: http://localhost:5000/_dozer/index

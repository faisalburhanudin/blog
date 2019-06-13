+++
title = "Parallel Task Python"
subtitle = ""

# Add a summary to display on homepage (optional).
summary = "Untuk menjalankan program secara parallel atau async, python menyediakan module concurrent.futures (python versi 3.2 keatas). Terdapat ThreadPoolExecutor untuk menjalankan di threads berbeda, dan ProcessPoolExecutor untuk menjalankan di proses yang berbeda."

date = 2019-06-13T10:45:19+07:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = []

# Is this a featured post? (true/false)
featured = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["python"]
categories = []

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""
+++


# Parallel task di python

Untuk menjalankan program secara parallel atau async, python menyediakan module `concurrent.futures` (python versi 3.2 keatas). Terdapat `ThreadPoolExecutor` untuk menjalankan di threads berbeda, dan `ProcessPoolExecutor` untuk menjalankan di proses yang berbeda.


`ThreadPoolExecutor` bagus digunakan untuk task yang prosesnya berjalan di `io` seperti network access, disk, dll

`ProcessPoolExecutor` bagus digunakan untuk task dengan proses yang berjalan di cpu. Seperti image resize, encoding, dll

## ThreadPoolExecutor

Contoh memanggil rest api secara bersamaan


```python
import requests
import time

def call_api():
    print('start request')
    r = requests.get('http://www.mocky.io/v2/5d014d8d3200002800f9db80?mocky-delay=2s')
    print('end request')
    return r.json()
```

Normal call

```python
start = time.time()
task1 = call_api()
task2 = call_api()

print(task1)
print(task2)
print('executed in: %.2f' % (time.time() - start))
```

output

    start request
    end request
    start request
    end request
    {'hello': 'world'}
    {'hello': 'world'}
    executed in: 5.15

Parallel

```python
from concurrent.futures import ThreadPoolExecutor

# using threadpool
start = time.time()
with ThreadPoolExecutor() as executor:
    task1 = executor.submit(call_api)
    task2 = executor.submit(call_api)
    print(task1.result())
    print(task2.result())
print('executed in: %.2f' % (time.time() - start))
```

output

    start request
    start request
    end requestend request
    
    {'hello': 'world'}
    {'hello': 'world'}
    executed in: 2.55


Bisa dilihat bahwa `call_api` dipanggil bersamaan saat menggunakan `ThreadPoolExecutor`

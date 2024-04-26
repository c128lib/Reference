---
layout: default
title: Books
---

# Book list 

This is list of known books about C128. Many of them can be found on [Archive.org](https://archive.org/).
Below a list of other Commodore machine, which can be useful too (especially when they talks about 6502 programming).

## Book list for C128

|Book|Name|
|-|-|
{% for singlefile in site.static_files %}{% if singlefile.path contains '/assets/images/books/c128/' %}|<img src="{{ singlefile.path }}" width="100">|{{ singlefile.basename }}|
{% endif %}{% endfor %}

## Book list for C64

|Book|Name|
|-|-|
{% for singlefile in site.static_files %}{% if singlefile.path contains '/assets/images/books/c64' %}|<img src="{{ singlefile.path }}" width="100">|{{ singlefile.basename }}|
{% endif %}{% endfor %}

## Book list for Vic20

|Book|Name|
|-|-|
{% for singlefile in site.static_files %}{% if singlefile.path contains '/assets/images/books/vic20' %}|<img src="{{ singlefile.path }}" width="100">|{{ singlefile.basename }}|
{% endif %}{% endfor %}

## Book list for C16/Plus4

|Book|Name|
|-|-|
{% for singlefile in site.static_files %}{% if singlefile.path contains '/assets/images/books/c16' %}|<img src="{{ singlefile.path }}" width="100">|{{ singlefile.basename }}|
{% endif %}{% endfor %}

# mF2C Documentation

Public documentation for the mF2C System

This documentation includes both the developer and user guide for the 
mF2C System developed in https://github.com/mF2C.

You can browse the rendered documentation on 
[https://mf2c-project.readthedocs.io/](https://mf2c-project.readthedocs.io/).

## Building

Install sphinx and the ReadTheDocs theme via pip, then run `make html`.

```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
make html
```

The documentation will be generated in `_build/html/index.html`.


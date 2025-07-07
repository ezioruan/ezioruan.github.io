---
title: "Getting Started with FastAPI for High-Performance APIs"
date: 2021-05-20 11:30:00
tags: Python, FastAPI, Web Development, API
---

When it comes to building APIs in Python, frameworks like Flask and Django have long been the go-to choices. However, a newer framework has rapidly gained popularity for its incredible performance and developer-friendly features: FastAPI. Built on top of Starlette and Pydantic, FastAPI makes it easy to create modern, fast, and well-documented APIs.

<!--more-->

### What is FastAPI?

FastAPI is a modern, high-performance web framework for building APIs with Python 3.6+ based on standard Python type hints. The key features are:

*   **Fast:** It is one of the fastest Python frameworks available, on par with NodeJS and Go.
*   **Fast to code:** Increase the speed to develop features by about 200% to 300%.
*   **Fewer bugs:** Reduce about 40% of human-induced errors.
*   **Intuitive:** Great editor support. Completion everywhere. Less time debugging.
*   **Easy:** Designed to be easy to use and learn. Less time reading docs.
*   **Short:** Minimize code duplication. Multiple features from each parameter declaration.
*   **Robust:** Get production-ready code. With automatic interactive documentation.

### Your First FastAPI Application

Let's create a simple API. First, you need to install FastAPI and an ASGI server, such as `uvicorn`.

```bash
pip install fastapi "uvicorn[standard]"
```

Now, create a file named `main.py`:

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

To run this application, use `uvicorn`:

```bash
uvicorn main:app --reload
```

The `--reload` flag tells `uvicorn` to automatically restart the server whenever you make changes to the code.

### Automatic Interactive Documentation

One of FastAPI's most beloved features is its automatic interactive API documentation. Once your application is running, you can navigate to two different URLs:

*   **/docs**: This provides interactive API documentation powered by Swagger UI.
*   **/redoc**: This provides alternative documentation powered by ReDoc.

These interfaces allow you to explore and test your API endpoints directly from your browser, without any extra configuration. This is possible because FastAPI uses the OpenAPI standard and Python type hints to generate the documentation automatically.

In the `read_item` function, we used type hints (`item_id: int` and `q: str | None`). FastAPI uses these hints not only for documentation but also for data validation and conversion. If you try to pass a non-integer value for `item_id`, FastAPI will automatically return a clear JSON error response.

### Pydantic for Data Validation

FastAPI's power is greatly enhanced by its integration with Pydantic. You can define data shapes as classes with type-annotated attributes. This gives you powerful data validation, serialization, and documentation all in one.

Let's define a simple `Item` model:

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool | None = None

@app.post("/items/")
def create_item(item: Item):
    return item
```

Now, when you make a POST request to `/items/`, FastAPI will expect a JSON body that matches the structure of the `Item` model. It will validate the types, check for required fields, and convert the JSON into a Python object that you can use in your code.

### Conclusion

FastAPI offers a compelling combination of speed, ease of use, and modern features. By leveraging Python type hints, it provides an exceptional developer experience with automatic data validation and interactive documentation. If you're starting a new API project in Python, FastAPI should be at the top of your list of frameworks to consider.

+++
title = "Sqlalchemy Polymorphic Inherintance"
subtitle = ""

# Add a summary to display on homepage (optional).
summary = "Pada implementasi database terkadang diperlukan sebuah tabel yang merupakan keturunan table lain contoh table users yang memiliki keturunan table employee dan table client, maka dalam implementasinya bisa dengan membuat struktur"

date = 2019-04-04T17:03:03+07:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = []

# Is this a featured post? (true/false)
featured = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = []
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
  caption = "Photo by Kevin Ku on Unsplash"

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""
+++
Pada implementasi database terkadang diperlukan sebuah tabel yang merupakan keturunan table lain contoh table `users` yang memiliki keturunan table `employee` dan table `client`, maka dalam implementasinya bisa dengan membuat struktur:

table `users`

```
-------------------------
| Field   | Type        | 
------------------------|
| id      | int(11)     |
| name    | varchar(50) |
| email   | varchar(50) |
| type    | varchar(10) |
-------------------------
```

table `employee`

```
----------------------------
| Field      | Type        |
---------------------------|
| id         | int(11)     |
| department | varchar(50) |
----------------------------
```

table `client`

```
-----------------------------
| Field    | Type           |
-----------------------------
| id       | int(11)        |
| address  | varchar(50)    |
-----------------------------
```

Dengan struktur table seperti itu maka bisa di representasikan menjadi model sqlalchemy menjadi seperti berikut:

preparing sqlalchemy connection, session, database dan base mode.

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///:memory')
Base = declarative_base()
Session = sessionmaker(bind=engine)
session = Session()Base = declarative_base()
```

model Users

```python
class Users(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    name = Column(String)
    email = Column(String)

    # digunakan sebagai discriminator
    type = Column(String)

    __mapper_args__ = {
        # jika type pada parent merupakan `user` maka object akan
        # menjadi instance dari class ini
        'polymorphic_identity': 'user',
        'polymorphic_on': type  # Parent model harus mengimplementasi ini
    }

    def __init__(self, name, email, _type='user'):
        self.name = name
        self.email = email
        self.type = _type

    def __repr__(self):
        return "<Users (name='%s', email='%s')>" % (self.name, self.email)
```

model employee

```python
class Employee(Users):
    __tablename__ = "employee"

    id = Column(None, ForeignKey("users.id"), primary_key=True)
    department = Column(String)

    __mapper_args__ = {
        # jika type pada parent merupakan `employee` maka object akan
        # menjadi instance dari class ini
        'polymorphic_identity': 'employee'
    }

    def __init__(self, name, email, department):
        super().__init__(name, email, 'employee')
        self.department = department

    def __repr__(self):
        return "<Employee (name='%s', email='%s', department='%s')>" % (
            self.name, self.email, self.department
        )
```

model client

```python
class Client(Users):
    __tablename__ = "client"

    id = Column(None, ForeignKey("users.id"), primary_key=True)
    address = Column(String)

    __mapper_args__ = {
        # jika type pada parent merupakan `client` maka object akan
        # menjadi instance dari class ini
        'polymorphic_identity': 'client'
    }

    def __init__(self, name, email, address):
        super().__init__(name, email, "client")
        self.address = address

    def __repr__(self):
        return "<Client (name='%s', email='%s', address='%s')>" % (
            self.name, self.email, self.address
        )
```

Saatnya testing.

```python
# create table
Base.metadata.create_all(engine)

user = Users("faisalburhanudin", "faisalburhanudin@hotmail.com")
session.add(user)

print(session.query(Users).all())

employee = Employee("gogil", "gogil@mail.com", "hrd")
session.add(employee)

print(session.query(Employee).all())

client = Client("client_name", "client@mail.com", "Klaten")
session.add(client)
session.add(client)

print(session.query(Client).all())
```
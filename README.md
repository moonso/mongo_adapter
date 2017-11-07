# Mongo Adapter

[![Build Status][travis-img]][travis-url]

A python implementation of a mongo adapter.

The idea is to make a base class that handles the connection to a mongod instance.
It is nice to handle the connection in the same way for different python projects that involves a mongo database.

The Mongo Adapter takes a client as init argument and then we connect to a database with `setup(database)`, forunately `mongo_adapter` can handle the client part as well.

**Example:**

```python
from mongo_adapter import MongoAdapter, get_client

db_name = 'test'
client = get_client()
adapter = MongoAdapter(client, db_name)

assert adapter.db_name == db_name
```

## installation

**git:**

```bash
git clone https://github.com/moonso/mongo_adapter
cd mongo_adapter
pip install --editable .
```

**pip:**
```bash
pip install mongo_adapter
```


## Intended usage

The intended usage is here illustrated with an example

```python
from mongo_adapter import MongoAdapter, get_client

class LibraryAdapter(MongoAdapter):
	def setup(self, database='library'):
		"""Overrides the basic setup method"""
		self.db = database

		self.books_collection = self.db.book
		self.user_collection = self.db.book
	
	def add_book(self, title, author):
		"""Add a book to the books collection"""
		result = self.books_collection.insert_one(
			{
				'title': title,
				'author': author
			}
		)
		return result

if __name__ == '__main__':
	client = get_client()
	adapter = LibraryAdapter(client, database='library')
	
	adapter.add_book('Moby Dick', 'Herman Melville')

```

## API

### Client

```python

def check_connection(client):
    """Check if the mongod process is running
    
    Args:
        client(MongoClient)
    
    Returns:
        bool
    """

def get_client(host='localhost', port=27017, username=None, password=None,
              uri=None, mongodb=None, timeout=20):
    """Get a client to the mongo database

    Args:
        host(str): Host of database
        port(int): Port of database
        username(str)
        password(str)
        uri(str)
        timeout(int): How long should the client try to connect

    Returns:
        client(pymongo.MongoClient)

    """

```

### Adapter

```python
class MongoAdapter(object):
    """Adapter for communicating with a mongo database"""
    def __init__(self, client=None, db_name=None):
        """
        Args:
            client(MongoClient)
            db_name(str)
        """
        self.client = client
        self.db = None
        self.db_name = None
        if (db_name and client):
            self.setup(database)
    
    def init_app(self, app):
        """Setup via Flask"""
        host = app.config.get('MONGO_HOST', 'localhost')
        port = app.config.get('MONGO_PORT', 27017)
        db_name = app.config['MONGO_DBNAME']
        LOG.info("connecting to database: %s:%s/%s", host, port, db_name)
        self.setup(db_name=db_name, db=app.extensions['pymongo']['MONGO'][1])
        
    
    def setup(self, db_name, db=None):
        """Setup connection to a database
        
        Args:
            db_name(str)
            db(pymongo.Database)
        """
        if db:
            self.db = db
            self.db_name = db.name
        else:
            self.db = self.client[db_name]
            self.db_name = db_name
```


[travis-url]: https://travis-ci.org/moonso/mongo_adapter
[travis-img]: https://img.shields.io/travis/moonso/mongo_adapter/master.svg?style=flat-square


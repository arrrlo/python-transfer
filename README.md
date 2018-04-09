<h1>Python Transfer</h1>

<p>An easy way to manipulate data using key-value databases like Redis.<br/>
It is designed to support a number of databases, but currently only Redis is supported.</p>

<h2>INSTALL (Python 3.x)</h2>

```bash
pip install git+git://github.com/arrrlo/python-transfer@master
```

<h2>Design</h2>

<p>There are an adapter class for every database.<br/>
After instantiating Python Transfer using certain adapter_name, we can manipulate the<br/>
data from key-value database just like dictionaries: python_transfer[key] = value</p>

<h2>Keys</h2>

<p>Keys are created using prefix, namespace and item.<br/>
Example: data:USERS:arrrlo:full_name<br/>
(data is prefix, USERS is namespace and arrrlo:full_name is item)</p>

<h2>Redis Adapter:</h2>

<h3>Connect to Redis using environment variables</h3>

```python
from python_transfer import Transfer, sent_env

os.environ['REDIS_HOST'] = 'localhost'
os.environ['REDIS_PORT'] = '6379'
os.environ['REDIS_DB'] = '0'

@sent_env('redis', 'HOST', 'REDIS_HOST')
@sent_env('redis', 'PORT', 'REDIS_PORT')
@sent_env('redis', 'DB', 'REDIS_DB')
class RedisTransfer(Transfer):

    def __init__(self, prefix, namespace):
        super().__init__(prefix=str(prefix), namespace=namespace, adapter_name='redis')
```

<h3>Store data</h3>

```python
rt = RedisTransfer()
rt['my_key'] = 'some_string' # redis: "SET" "data:my_key" "some_string"

rt = RedisTransfer(namespace='my_namespace')
rt['my_key'] = 'some_string' # redis: "SET" "data:my_name_space:my_key" "some_string"

rt = RedisTransfer(prefix='my_prefix', namespace='my_namespace')
rt['my_key'] = 'some_string' # redis: "SET" "my_prefix:my_name_space:my_key" "some_string"
```

<h3>Connect to Redis using class parameters</h3>

```python
class RedisTransfer(Transfer):

    def __init__(self, prefix, namespace, host, port, db):
        super().__init__(prefix=str(prefix), namespace=namespace, adapter_name='redis')

        self.set_env('HOST', host)
        self.set_env('PORT', port)
        self.set_env('DB', db)
```

<h3>Store data</h3>

```python
rt = RedisTransfer(prefix='my_prefix', namespace='my_namespace', host='localhost', port=6379, db=0)
rt['my_key'] = 'some_string' # redis: "SET" "my_prefix:my_name_space:my_key" "some_string"
```

<h3>Fetch data</h3>

```python
my_var = rt['my_key'] # redis: "GET" "my_prefix:my_name_space:my_key"
```

<h3>Delete data</h3>

```python
del rt['my_key'] # redis: "DEL" "my_prefix:my_name_space:my_key"
```

<h3>Other data types</h3>

```python
rt['my_key_1'] = [1,2,3,4] # redis: "RPUSH" "my_prefix:my_name_space:my_key" "1" "2" "3" "4"
rt['my_key_2'] = {'foo': 'bar'} # redis: "HMSET" "my_prefix:my_name_space:my_key" "foo" "bar"

my_var_1 = rt['my_key_1'] # redis: "LRANGE" "my_prefix:my_name_space:my_key_1" "0" "-1"
my_var_2 = rt['my_key_2'] # redis: "HGETALL" "my_prefix:my_name_space:my_key_2"
```

<h3>Using redis pipeline (multiple commands execution, only for set and delete)</h3>

```python
with rt:
    rt['my_key_1'] = 'some_string'
    rt['my_key_2'] = [1,2,3,4]
    rt['my_key_3'] = {'foo': 'bar'}
```

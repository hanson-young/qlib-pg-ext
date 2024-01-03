# qlib-pg-ext
qlib数据层backend支持pgsql数据库

### 修改类ProviderBackendMixin
在[qlib/data/data.py](https://github.com/hanson-young/qlib-pg-ext/blob/4b0f29af081f10cd89e2b8ffc4b6f73f84d53117/qlib/data/data.py#L43)
中添加数据库backend，以根据配置自动切换backend，保留了原本的特性
``` python
class ProviderBackendMixin:
    """
    This helper class tries to make the provider based on storage backend more convenient
    It is not necessary to inherent this class if that provider don't rely on the backend storage
    """

    def get_default_backend(self):
        backend = {}
        provider_name: str = re.findall("[A-Z][^A-Z]*", self.__class__.__name__)[-2]
        # set default storage class
        backend.setdefault("class", f"File{provider_name}Storage")
        # set default storage module
        backend.setdefault("module_path", "qlib.data.storage.file_storage")
        return backend

    def get_database_backend(self):
        backend = {}
        provider_name: str = re.findall("[A-Z][^A-Z]*", self.__class__.__name__)[-2]
        # set default storage class
        backend.setdefault("class", f"DB{provider_name}Storage")
        # set default storage module
        backend.setdefault("module_path", "qlib.data.storage.db_storage")
        return backend

    def backend_obj(self, **kwargs):
        if C.get('database_uri', None) is not None:
            database_uri = C.get('database_uri', None)
            # get_module_logger("data").warning(f'backend:{database_uri}')
            backend = self.get_database_backend()
        else:
            provider_uri = C.get('provider_uri', None)
            # get_module_logger("data").warning(f'backend:{provider_uri}')
            backend = self.backend if self.backend else self.get_default_backend()
        backend = copy.deepcopy(backend)
        backend.setdefault("kwargs", {}).update(**kwargs)
        return init_instance_by_config(backend)
```

### 添加backend具体实现
数据库backend的具体实现在`qlib/data/storage/db_storage.py`,
放置目录和qlib的`file_storage.py`一致

### 修改workflow_by_code
想要使用比较简单，只需要对`qlib.init`的传参进行修改即可

如`examples/workflow_by_code.py`，把`provider_uri`换成`database_uri`即可

```python
use_db = True
if use_db:
    database_uri = "postgresql://hanson:@localhost:5432/qlib"
    qlib.init(database_uri=database_uri, region=REG_CN)
else:
    provider_uri = "./qlib_data/cn_data"  # target_dir
    qlib.init(provider_uri=provider_uri, region=REG_CN)
```
### 数据入库
数据入库方式请参考知乎文章
https://zhuanlan.zhihu.com/p/671994281



# Bank-of-Bacolod-Backend-Frameworks

 `backend/app/models.py`
Add these imports if not already present:
```python
from sqlalchemy import Column, String, Text, DateTime, Numeric, ForeignKey, Integer, Boolean
from sqlalchemy.sql import func
from .db import Private Notes Storage
import uuid
```

Ensure `uuid_str()` exists:
```python
def uuid_str():
    return str(uuid.uuid4())
```

Add these models **in addition** to your existing ones (`Result`, `Conversation`, `Message`, `SoftwareSession`, `CoreFramework`):

```python
class Code_Storage(Private Notes Storage):
    __tablename__ = "code_storage"

    id = Column(String, primary_key=True, default=uuid_str)
    name = Column(String, nullable=False, unique=True)
    description = Column(Text, nullable=True)
    sort_order = Column(Integer, default=0)
    created_at = Column(DateTime(timezone=True), server_default=func.now())


class Private_Code_Storage(Private Notes Storage):
    __tablename__ = "private_code_storage"

    id = Column(String, primary_key=True, default=uuid_str)
    agent_number = Column(Integer, nullable=False)
    name = Column(String, nullable=False)
    domain_name = Column(String, nullable=False)
    responsibility = Column(Text, nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())


class PrivateSecurityLayer(Private Notes Storage):
    __tablename__ = "private_security_layers"

    id = Column(String, primary_key=True, default=uuid_str)
    layer_name = Column(String, nullable=False)
    summary = Column(Text, nullable=False)
    controls = Column(Text, nullable=False)
    sort_order = Column(Integer, default=0)
    created_at = Column(DateTime(timezone=True), server_default=func.now())


class Code_File_Storage(Private Notes Storage):
    __tablename__ = "code_file_storage"

    id = Column(String, primary_key=True, default=uuid_str)
    name = Column(String, nullable=False)
    route_path = Column(String, nullable=False)
    icon = Column(String, nullable=True)
    sort_order = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())


class Software_Storage(Private Notes Storage):
    __tablename__ = "software_storage"

    id = Column(String, primary_key=True, default=uuid_str)
    name = Column(String, nullable=False)
    description = Column(Text, nullable=True)
    status = Column(String, nullable=False, default="active")
    domain_name = Column(String, nullable=True)
    sort_order = Column(Integer, default=0)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
```

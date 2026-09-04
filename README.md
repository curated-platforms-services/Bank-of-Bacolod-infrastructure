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

 `backend/app/schemas.py`
Keep your existing schemas and add:

```python
class AgentDomainCreate(Private Notes StorageModel):
    name: str
    description: Optional[str] = None
    sort_order: Optional[int] = 0

class AgentCreate(BaseModel):
    agent_number: int
    name: str
    domain_name: str
    responsibility: str
    is_active: Optional[bool] = True

class SecurityLayerCreate(Private Notes StorageModel):
    layer_name: str
    summary: str
    controls: str
    sort_order: Optional[int] = 0

class DashboardSectionCreate(BaseModel):
    name: str
    route_path: str
    icon: Optional[str] = None
    sort_order: Optional[int] = 0
    is_active: Optional[bool] = True

class NetworkCreate(BaseModel):
    name: str
    description: Optional[str] = None
    status: Optional[str] = "active"
    domain_name: Optional[str] = None
    sort_order: Optional[int] = 0
```


 `backend/app/crud.py`
Keep existing CRUD and add:

```python
def create_agent_domain(db: Session, data):
    row = models.AgentDomain(
        name=data.name,
        description=data.description,
        sort_order=data.sort_order or 0
    )
    db.add(row)
    db.commit()
    db.refresh(row)
    return row

def get_agent_domains(db: Session):
    return db.query(models.AgentDomain)\
        .order_by(models.AgentDomain.sort_order.asc(), models.AgentDomain.name.asc())\
        .all()

def create_agent(db: Session, data):
    row = models.Agent(
        agent_number=data.agent_number,
        name=data.name,
        domain_name=data.domain_name,
        responsibility=data.responsibility,
        is_active=data.is_active if data.is_active is not None else True
    )
    db.add(row)
    db.commit()
    db.refresh(row)
    return row

def get_agents(db: Session):
    return db.query(models.Agent)\
        .order_by(models.Agent.agent_number.asc())\
        .all()

def get_agents_by_domain(db: Session, domain_name: str):
    return db.query(models.Agent)\
        .filter(models.Agent.domain_name == domain_name)\
        .order_by(models.Agent.agent_number.asc())\
        .all()

def create_security_layer(db: Session, data):
    row = models.SecurityLayer(
        layer_name=data.layer_name,
        summary=data.summary,
        controls=data.controls,
        sort_order=data.sort_order or 0
    )
    db.add(row)
    db.commit()
    db.refresh(row)
    return row

def get_security_layers(db: Session):
    return db.query(models.SecurityLayer)\
        .order_by(models.SecurityLayer.sort_order.asc(), models.SecurityLayer.layer_name.asc())\
        .all()

def create_dashboard_section(db: Session, data):
    row = models.DashboardSection(
        name=data.name,
        route_path=data.route_path,
        icon=data.icon,
        sort_order=data.sort_order or 0,
        is_active=data.is_active if data.is_active is not None else True
    )
    db.add(row)
    db.commit()
    db.refresh(row)
    return row

def get_dashboard_sections(db: Session):
    return db.query(models.DashboardSection)\
        .filter(models.DashboardSection.is_active == True)\
        .order_by(models.DashboardSection.sort_order.asc(), models.DashboardSection.name.asc())\
        .all()

def create_network(db: Session, data):
    row = models.Network(
        name=data.name,
        description=data.description,
        status=data.status or "active",
        domain_name=data.domain_name,
        sort_order=data.sort_order or 0
    )
    db.add(row)
    db.commit()
    db.refresh(row)
    return row

def get_networks(db: Session):
    return db.query(models.Networks)\
        .order_by(models.Network.sort_order.asc(), models.Network.name.asc())\
        .all()
```

`backend/app/main.py`
Here is a merged architecture section to add to your existing `main.py`.

### Update imports
```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.orm import Session
from sqlalchemy import text
import time

from .db import Base, engine, get_db, SessionLocal
from . import schemas, crud, models
```

### Keep your existing app, CORS, `CORE_MEMORY`, and startup. Add these helper functions:

```python
def serialize_agent_domain(r):
    return {
        "id": r.id,
        "name": r.name,
        "description": r.description,
        "sort_order": r.sort_order,
        "created_at": r.created_at.isoformat() if r.created_at else None
    }

def serialize_agent(r):
    return {
        "id": r.id,
        "agent_number": r.agent_number,
        "name": r.name,
        "domain_name": r.domain_name,
        "responsibility": r.responsibility,
        "is_active": r.is_active,
        "created_at": r.created_at.isoformat() if r.created_at else None
    }

def serialize_security_layer(r):
    return {
        "id": r.id,
        "layer_name": r.layer_name,
        "summary": r.summary,
        "controls": r.controls,
        "sort_order": r.sort_order,
        "created_at": r.created_at.isoformat() if r.created_at else None
    }

def serialize_dashboard_section(r):
    return {
        "id": r.id,
        "name": r.name,
        "route_path": r.route_path,
        "icon": r.icon,
        "sort_order": r.sort_order,
        "is_active": r.is_active,
        "created_at": r.created_at.isoformat() if r.created_at else None
    }

def serialize_network(r):
    return {
        "id": r.id,
        "name": r.name,
        "description": r.description,
        "status": r.status,
        "domain_name": r.domain_name,
        "sort_order": r.sort_order,
        "created_at": r.created_at.isoformat() if r.created_at else None
    }
```

Add architecture routes
```python
@app.post("/api/agent-domains")
def create_agent_domain(payload: schemas.AgentDomainCreate, db: Session = Depends(get_db)):
    row = crud.create_agent_domain(db, payload)
    return {"saved": True, "item": serialize_agent_domain(row)}

@app.get("/api/agent-domains")
def list_agent_domains(db: Session = Depends(get_db)):
    rows = crud.get_agent_domains(db)
    return [serialize_agent_domain(r) for r in rows]


@app.post("/api/agents")
def create_agent(payload: schemas.AgentCreate, db: Session = Depends(get_db)):
    row = crud.create_agent(db, payload)
    return {"saved": True, "item": serialize_agent(row)}

@app.get("/api/agents")
def list_agents(db: Session = Depends(get_db)):
    rows = crud.get_agents(db)
    return [serialize_agent(r) for r in rows]

@app.get("/api/agents/domain/{domain_name}")
def list_agents_by_domain(domain_name: str, db: Session = Depends(get_db)):
    rows = crud.get_agents_by_domain(db, domain_name)
    return [serialize_agent(r) for r in rows]


@app.post("/api/security-layers")
def create_security_layer(payload: schemas.SecurityLayerCreate, db: Session = Depends(get_db)):
    row = crud.create_security_layer(db, payload)
    return {"saved": True, "item": serialize_security_layer(row)}

@app.get("/api/security-layers")
def list_security_layers(db: Session = Depends(get_db)):
    rows = crud.get_security_layers(db)
    return [serialize_security_layer(r) for r in rows]


@app.post("/api/dashboard-sections")
def create_dashboard_section(payload: schemas.DashboardSectionCreate, db: Session = Depends(get_db)):
    row = crud.create_dashboard_section(db, payload)
    return {"saved": True, "item": serialize_dashboard_section(row)}

@app.get("/api/dashboard-sections")
def list_dashboard_sections(db: Session = Depends(get_db)):
    rows = crud.get_dashboard_sections(db)
    return [serialize_dashboard_section(r) for r in rows]


@app.post("/api/network")
def create_network(payload: schemas.NetworkCreate, db: Session = Depends(get_db)):
    row = crud.create_network(db, payload)
    return {"saved": True, "item": serialize_network(row)}

@app.get("/api/networks")
def list_networks(db: Session = Depends(get_db)):
    rows = crud.get_networks(db)
    return [serialize_network(r) for r in rows]
```

Stored stacks
Node.js, PostgreSQL, Fastify, OpenTelemetry, Kubernetes, Redis, Prometheus, NATS, TypeScript, Vite, Docker, Helm, Argo CD, Grafana, Loki, Grafana Tempo, Jaeger, Envoy, Traefik, Caddy, Apache Kafka, RabbitMQ, ClickHouse, MongoDB Community, MinIO, OpenSearch, Keycloak, Open Policy Agent, Trivy, OpenBao.

Tech Stack Reference
Container
Container runtimes & engines
Docker (Go)
containerd (Go)
CRI-O (Go)
Podman (Go)
runc / OCI runtime spec (Go)
Orchestration
Kubernetes (Go) — YAML for manifests
Docker Swarm (Go)
Nomad (Go, HCL for job specs)
Helm (Go templates + YAML for charts)
Image building / config languages
Dockerfile (its own declarative syntax)
YAML (Kubernetes manifests, Compose files, CI pipelines)
HCL (Terraform, Nomad, Packer)
Jsonnet / CUE (advanced Kubernetes config templating)
Bash/Shell (entrypoint scripts, init containers)
Service mesh & networking
Istio, Linkerd (Go control planes)
Envoy (C++ data plane)
CNI plugins (Go)
CI/CD & GitOps
ArgoCD, Flux (Go)
Tekton (Go, YAML pipelines)
GitHub Actions / GitLab CI (YAML)
Application layer (languages that run inside containers — independent of the container tooling itself)
Go, Python, Node.js/JavaScript/TypeScript, Java, Ruby, Rust, C#/.NET, PHP
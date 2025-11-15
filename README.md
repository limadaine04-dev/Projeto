# Projeto

git checkout -b rest

from fastapi import APIRouter, HTTPException
from models.usuario import Usuario
from database import SessionLocal
from schemas.usuario_schema import UsuarioCreate, UsuarioResponse

router = APIRouter(prefix="/usuarios", tags=["Usuários"])

# Criar
@router.post("/", response_model=UsuarioResponse)
def criar_usuario(usuario: UsuarioCreate):
    db = SessionLocal()
    novo_usuario = Usuario(**usuario.dict())
    db.add(novo_usuario)
    db.commit()
    db.refresh(novo_usuario)
    return novo_usuario

# Listar
@router.get("/", response_model=list[UsuarioResponse])
def listar_usuarios():
    db = SessionLocal()
    return db.query(Usuario).all()

# Atualizar
@router.put("/{usuario_id}", response_model=UsuarioResponse)
def atualizar_usuario(usuario_id: int, dados: UsuarioCreate):
    db = SessionLocal()
    usuario = db.query(Usuario).filter(Usuario.id == usuario_id).first()

    if not usuario:
        raise HTTPException(status_code=404, detail="Usuário não encontrado")

    for campo, valor in dados.dict().items():
        setattr(usuario, campo, valor)

    db.commit()
    db.refresh(usuario)
    return usuario

# Apagar
@router.delete("/{usuario_id}")
def deletar_usuario(usuario_id: int):
    db = SessionLocal()
    usuario = db.query(Usuario).filter(Usuario.id == usuario_id).first()

    if not usuario:
        raise HTTPException(status_code=404, detail="Usuário não encontrado")

    db.delete(usuario)
    db.commit()
    return {"detail": "Usuário removido com sucesso"}



from fastapi import APIRouter, HTTPException
from models.cliente import Cliente
from database import SessionLocal
from schemas.cliente_schema import ClienteCreate, ClienteResponse

router = APIRouter(prefix="/clientes", tags=["Administradores"])

# Criar
@router.post("/", response_model=ClienteResponse)
def criar_admin(cliente: ClienteCreate):
    db = SessionLocal()
    novo = Cliente(**cliente.dict())
    db.add(novo)
    db.commit()
    db.refresh(novo)
    return novo

# Listar
@router.get("/", response_model=list[ClienteResponse])
def listar_admins():
    db = SessionLocal()
    return db.query(Cliente).all()

# Atualizar
@router.put("/{cliente_id}", response_model=ClienteResponse)
def atualizar_admin(cliente_id: int, dados: ClienteCreate):
    db = SessionLocal()
    admin = db.query(Cliente).filter(Cliente.id == cliente_id).first()

    if not admin:
        raise HTTPException(status_code=404, detail="Administrador não encontrado")

    for campo, valor in dados.dict().items():
        setattr(admin, campo, valor)

    db.commit()
    db.refresh(admin)
    return admin

# Apagar
@router.delete("/{cliente_id}")
def deletar_admin(cliente_id: int):
    db = SessionLocal()
    admin = db.query(Cliente).filter(Cliente.id == cliente_id).first()

    if not admin:
        raise HTTPException(status_code=404, detail="Administrador não encontrado")

    db.delete(admin)
    db.commit()
    return {"detail": "Administrador removido com sucesso"}



uvicorn main:app --reload

class UsuarioCreate(BaseModel):
    nome: str = Field(..., example="Maria Souza")
    email: str = Field(..., example="maria@gmail.com")


from fastapi import FastAPI
from controllers.usuario_controller import router as usuario_router
from controllers.clientes_controller import router as clientes_router

app = FastAPI(title="API do Projeto", version="1.0")

app.include_router(usuario_router)
app.include_router(clientes_router)


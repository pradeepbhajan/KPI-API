# app/models.py
from sqlalchemy import Column, Integer, String
from app.database import Base

class Form(Base):
    __tablename__ = "forms"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False)
    email = Column(String, nullable=False)
    message = Column(String, nullable=True)

# app/schemas.py
from pydantic import BaseModel, EmailStr

class FormBase(BaseModel):
    name: str
    email: EmailStr
    message: str | None = None

class FormCreate(FormBase):
    pass

class Form(FormBase):
    id: int

    class Config:
        orm_mode = True

# app/crud.py
from sqlalchemy.orm import Session
from app import models, schemas

def get_forms(db: Session):
    return db.query(models.Form).all()

def create_form(db: Session, form: schemas.FormCreate):
    db_form = models.Form(**form.dict())
    db.add(db_form)
    db.commit()
    db.refresh(db_form)
    return db_form

# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from dotenv import load_dotenv

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# main.py
from fastapi import FastAPI, HTTPException, Depends
from sqlalchemy.orm import Session
from app import models, schemas, crud, database
from app.database import engine, SessionLocal
from fastapi.middleware.cors import CORSMiddleware
import os

models.Base.metadata.create_all(bind=engine)

app = FastAPI()

origins = ["http://localhost:3000"]
app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/forms", response_model=list[schemas.Form])
def get_forms(db: Session = Depends(get_db)):
    return crud.get_forms(db)

@app.post("/forms", response_model=schemas.Form)
def create_form(form: schemas.FormCreate, db: Session = Depends(get_db)):
    return crud.create_form(db, form)


# README.md

# KPA FastAPI Backend

## 📌 Project Overview (परियोजना अवलोकन)
This backend application is built using **FastAPI**, **PostgreSQL**, and **SQLAlchemy**. It allows a form with name, email, and message to be submitted and retrieved.

यह बैकएंड एप्लिकेशन FastAPI, PostgreSQL और SQLAlchemy का उपयोग करता है। इसमें एक फॉर्म है जिसमें उपयोगकर्ता नाम, ईमेल और मैसेज सबमिट कर सकते हैं।

---

## 🏗️ Tech Stack (प्रौद्योगिकी स्टैक)
- FastAPI 🔥
- PostgreSQL 🐘
- SQLAlchemy ⚙️
- Pydantic ✅
- Python Dotenv 📁

---

## 🔧 Setup Instructions (सेटअप निर्देश)

### 1. Clone Repository
```bash
git clone https://github.com/pradeepbhajan/kpa-fastapi-backend.git
cd kpa-fastapi-backend
```

### 2. Create & Activate Virtual Environment
```bash
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Create `.env` File
```
DATABASE_URL=postgresql://youruser:yourpass@localhost:5432/formdb
```

### 5. Run Server
```bash
uvicorn main:app --reload
```

Visit: http://127.0.0.1:8000/docs (Swagger UI)

---

## 📬 API Endpoints

| Method | Endpoint   | Description               |
|--------|------------|---------------------------|
| GET    | `/forms`   | Get all submitted forms   |
| POST   | `/forms`   | Submit a new form         |

---

## 🧪 Sample JSON Request
```json
{
  "name": "Pradeep Kumar",
  "email": "pradeep@example.com",
  "message": "This is a test form."
}
```

---

## 📂 Folder Structure
```
kpa-fastapi-backend/
├── app/
│   ├── crud.py
│   ├── database.py
│   ├── models.py
│   └── schemas.py
├── main.py
├── .env
├── requirements.txt
└── README.md
```

---

## ✍️ Author
- [Pradeep Kumar](https://github.com/pradeepbhajan)

---

## 📃 License
This project is licensed under the MIT License.

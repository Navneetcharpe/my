# json - get push add update delete

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List

app = FastAPI()

# Pydantic model for tea
class Tea(BaseModel):
    id: int
    name: str
    origin: str

# In-memory storage
teas: List[Tea] = []

@app.get("/")
def read_root():
    return {"message": "welcome my office"}

@app.get("/teas", response_model=List[Tea])
def get_teas():
    return teas

@app.post("/teas", response_model=Tea, status_code=201)
def add_tea(tea: Tea):
    # Ensure unique ID
    if any(t.id == tea.id for t in teas):
        raise HTTPException(status_code=400, detail="Tea with this ID already exists")
    teas.append(tea)
    return tea

@app.put("/teas/{tea_id}", response_model=Tea)
def update_tea(tea_id: int, updated_tea: Tea):
    for index, existing in enumerate(teas):
        if existing.id == tea_id:
            teas[index] = updated_tea
            return updated_tea
    raise HTTPException(status_code=404, detail="Tea not found")

@app.delete("/teas/{tea_id}", response_model=Tea)
def delete_tea(tea_id: int):
    for index, existing in enumerate(teas):
        if existing.id == tea_id:
            return teas.pop(index)
    raise HTTPException(status_code=404, detail="Tea not found")

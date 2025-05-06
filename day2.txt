from fastapi import FastAPI,status,HTTPException,APIRouter,Depends
from random import randrange
from .import schemas,models
from sqlalchemy.orm import Session 
from .database import engine,get_connection

# creating fast api object

router= APIRouter(tags=["User BOA Application"])
# dummy records
user_data=[]

# common find method to search the record (will help in search,update and delete)
def findById(id):
    for i,p in enumerate(user_data):
        if p['id'] == id:
            print(i,p)
            return i

#test db connection
@router.get('/test')
def conntest(db:Session=Depends(get_connection)):
    return {"status":"connected"}

# load the users from the server
@router.get('/loadusers')
def loadusers(db:Session=Depends(get_connection)):
    posts=db.query(models.UserApp).all()
    return posts

#load specific record
@router.get('/loaduser/{name}')
def loadUserById(name:str,db:Session=Depends(get_connection)):
     post= db.query(models.UserApp).filter(models.UserApp.uname== name).first()
     return {'user found ': post}

# delete the found record
@router.delete('/deleteuser/{name}')
def deleteUserById(name:str,db:Session=Depends(get_connection)):
    post= db.query(models.UserApp).filter(models.UserApp.uname== name)
    if post.first() ==None:
     
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,detail= "OOPS, user not found"
        )
    post.delete(synchronize_session=False)
    db.commit()
    return {'message ': 'user found and deleted '}


# update the found record with the new records
@router.put('/updateuser/{name}')
def updateUserById(name:str,newuser:schemas.User,db:Session=Depends(get_connection)):
    post= db.query(models.UserApp).filter(models.UserApp.uname== name)
    if post.first() ==None:
     
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,detail= "OOPS, user not found"
        )
    post.update(newuser.model_dump(),synchronize_session=False)
    db.commit()
    return {'message ': 'user found and updted '}

# add a new record
@router.post('/create', status_code=status.HTTP_201_CREATED)
def addUser(payload: schemas.User,db:Session=Depends(get_connection)):

    
    postdata=models.UserApp(**payload.dict())
    print(postdata)
    db.add(postdata)
    db.commit()
    db.refresh(postdata)
    return {'userdata':postdata}

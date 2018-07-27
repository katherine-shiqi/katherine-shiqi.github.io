---
layout:     post
title:      Tricks for Python
subtitle:   Multiprocessing & SQL Python Connection
date:       2018-07-27
author:     Katherine Li
header-img: img/post-bg-miui6.jpg
catalog: true
tags:
    - project
---
I wanted to share some tricks or methods that I used for python projects. They are simple but I hope they can save your time from googling. 

## 1. Reduce the running time by multiprocessing/multithreading
 

The concept is to allocate tasks to different processors running in parallel if the algorithm is not sequential. 

 

import multiprocessing

def function1(q1,parameters):
    #write your code
    q1.put(result)

def function2(q2,parameters):
    #write your code
    q2.put(result)

q1=multiprocessing.Manager().Queue()
q2=multiprocessing.Manager().Queue()
p1=multiprocessing.Process(target=f1,args=(q1,parameters,))
p2=multiprocessing.Process(target=f2,args=(q2,parameters,))
p1.start()
p2.start()
p1.join()
p2.join()

while q1.empty() is False:
    result1=q1.get()
  
while q2.empty() is False:
    result2=q2.get()
Pay attention to the queue size limit which I spent some time to figure out among the poorly written documentation.

 

As with threads, a common use pattern for multiple processes is to divide a job up among several workers to run in parallel. Effective use of multiple processes usually requires some communication between them, so that work can be divided and results can be aggregated. 

 

On the multiprocessing documentation, it guides us to use multiprocessing.Queue() to store output.  But there is a warning on the last page stating that the  Queue has the size limit. If the output you put to the queue exceeds the limit, it will cause a deadlock of the code (code will not end and generate output). The trick is to use multiprocessing.Manager.Queue().

 

For a loop, if it iterates through a large data frame with millions of rows, you can break it into several trunks and call different processors to run it.

def function (a,q1,q2,q3,parameters):
   if a==1:
      start,end=None, int(len(df)/3)
   elif a==2:
      start,end=int(len(df)/3),int(len(df)2/3)
   else:
      start,end=int(len(df)2/3),None

   for row in df[start:end].iterrows():
      #write your code here
   result=#get your result

   if a==1:
       q1.put(result)
   elif a==2:
       q2.put(result)
   else:
       q3.put(result)



q1=multiprocessing.Manager().Queue()
q2=multiprocessing.Manager().Queue()
q3=multiprocessing.Manager().Queue()
p1=multiprocessing.Process(target=f1,args=(1,q1,q2,q3,parameters))
p2=multiprocessing.Process(target=f1,args=(2,q1,q2,q3,parameters))
p3=multiprocessing.Process(target=f1,args=(3,q1,q2,q3,parameters))

p1.start()
p2.start()
p3.start()

while q1.empty() is False:
    result=q1.get()
while q2.empty() is False:
    result=pd.concat([result,q2.get()],axis=0,ignore_index=True)
while q3.empty() is False:
    result=pd.concat([result,q3.get()],axis=0,ignore_index=True)
    
 

Please let me know if you have better methods to conduct multiprocessing.

 

## 2. Connecting python with SQL database (get the data and push the data back)
 

There are a lot of tutorials online about how to do the configuration to pull data from SQL server to your python program. I listed here for your information. 

http://thelaziestprogrammer.com/sharrington/databases/connecting-to-sql-server-with-sqlalchemy

 

(1) Install
brew install unixodbc 
brew install freetds --with-unixodbc

(2) ODBC
vim /usr/local/etc/odbc.ini
sudo vim /etc/odbc.ini

[DSN]
Driver = FreeTDS
Server = #input your hostname or IP
Port = 1433
Database = #input your database name

(3)OBDCINST
--get the directory by running: odbcinst -j
sudo vim odbcinst.ini 
[FreeTDS]
Driver=/usr/local/lib/libtdsodbc.so
Setup=/usr/local/lib/libtdsS.so
FileUsage=1
UsageCount=1

(4)CONNECT
isql DSN name password

--------Go to Python------
from sqlalchemy import create_engine
import pandas as ps
engine = create_engine("mssql+pyodbc://name:password@DSN")
conn = engine.connect()
data = pd.read_sql_query('select * from table', conn)
data.to_sql('tablename', con=engine, if_exists='append', index=False) 

## 3. Running code on AWS GPU

(1) Key

Download the public key pub_key.pem

mv /Users/katherine/Desktop/pub_key.pem . security import pub_key.pem -k ~/Library/Keychains/login.keychain

(2) Login

chmod 400 pub_key.pem
ssh -i pub_key.pem ec2-user@XXX.compute-1.amazonaws.com

(3) Move folder (cplai) to ec2
scp -i pub_key.pem -r /Users/katherine/cplai xxx.compute-1.amazonaws.com:~

(4) Create Virtual environment
conda create -n env python=3.5
source activate env

(5) Save output to local desktop
scp -i pub_key.pem -r xxx.compute-1.amazonaws.com:~/cplai/Output /Users/Katherine/Desk





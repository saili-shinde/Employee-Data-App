# Employee-Data-App
Python Based project: used tkinter module for frontend, Mongodb for backend(storage). It takes ID, Name, Salary from user and store it in data base, user can change/update his name or salary or delete his data from data base. this app also show top 5 salaried employees graph 

## Code:

```
from tkinter import *
from tkinter.messagebox import *
from tkinter.scrolledtext import *
from pymongo import *
import requests
import matplotlib.pyplot as plt

#BACKEND

#main to add window
def f1():
	mw.withdraw()
	aw.deiconify()
  
#Add data to database
def f1_1():
	con=None
	try:
		#STORAGE
		con=MongoClient("mongodb://localhost:27017")
		db=con["ssnov22"]
		coll=db["employee"]

		try:
			eno=int(aw_ent_eno.get())
			if eno<=0:
				raise Exception("id should not be -ve or zero")
		except Exception:
			showerror("issue","Id should not be empty/non integer/-ve/zero")
			return

		try:
			name=str(aw_ent_name.get())
			if name.isalpha()==False or len(name)>2 or len(name)==0:
				raise Exception("name should contain alphabates only and length should not be greater than 2")
		except Exception:
			showerror("issue","name should not be non alphabetical/empty/longer than 2 aplphabates")
			return

		try:
			salary=float(aw_ent_salary.get())
			if salary<8000.0 or len(str(salary))==0:
				raise Exception("salary should be greater than 8000 and not empty")
		except Exception:
			showerror("issue","Salary should not be less than 8000/empty/non integer")
			return

		count=coll.count_documents({"_id":eno})
		if count==1:
			showerror("issue","record already exists")
		else:
			data={"_id":eno,"name":name,"salary":salary}
			coll.insert_one(data)
			showinfo("Success","record saved")
	except Exception as e:
		showerror("issue",str(e))
	finally:
		if con is not None:
			con.close()
		aw_ent_eno.delete(0,END)
		aw_ent_name.delete(0,END)
		aw_ent_salary.delete(0,END)
		aw_ent_eno.focus()
    
#add to main window
def f1_2():
	aw.withdraw()
	mw.deiconify()
  
#main to view window
def f2():
	mw.withdraw()
	vw.deiconify()
	vw_st_data.delete(1.0,END)
	con=None
	try:
		#STORAGE
		con=MongoClient("mongodb://localhost:27017/")
		db=con["ssnov22"]
		coll=db["employee"]
		data=coll.find().sort("_id",1)
		info=""
		for d in data:
			info=info + "eno= "+str(d["_id"]) + "\t name= "+str(d["name"]) + "\t Salary= " + str(d["salary"])+"\n"
		vw_st_data.insert(INSERT,info)
	except Exception as e:
		showerror("issue",str(e))
	finally:
		if con is not None:
			con.close()
      
#view to main window
def f2_1():
	vw.withdraw()
	mw.deiconify()
  
#main to update window
def f3():
	mw.withdraw()
	uw.deiconify()

#update
def f3_1():
	con=None
	try:
		#STORAGE
		con=MongoClient("mongodb://localhost:27017")
		db=con["ssnov22"]
		coll=db["employee"]
		try:
			eno=int(uw_ent_eno.get())
			if eno<=0:
				raise Exception("id should not be -ve or zero")
		except Exception:
			showerror("issue","Id should not be empty/non integer/-ve/zero")
			return

		try:
			name=str(uw_ent_name.get())
			if name.isalpha()==False or len(name)>2:
				raise Exception("name should contain alphabates only and length should not be greater than 2")
		except Exception:
			showerror("issue","name should not be non alphabetical/empty/longer than 2 aplphabates")
			return

		try:
			salary=float(uw_ent_salary.get())
			if salary<8000.0 or len(str(salary))==0:
				raise Exception("salary should be greater than 8000 and not empty")
		except Exception:
			showerror("issue","Salary should not be less than 8000/empty/non integer")
			return

		count=coll.count_documents({"_id":eno})
		if count==1:
			coll.update_one({"_id":eno},{"$set":{"name":name,"salary":salary}})
			showinfo("Success","record updated")

		else:
			showerror("issue","record does not exists")
	except Exception as e:
		showerror("issue",str(e))
	finally:
		if con is not None:
			con.close()
		uw_ent_eno.delete(0,END)
		uw_ent_name.delete(0,END)
		uw_ent_salary.delete(0,END)
		uw_ent_eno.focus()

#update to main
def f3_2():
	uw.withdraw()
	mw.deiconify()

#update to main
def f4():
	mw.withdraw()
	dw.deiconify()

#delete
def f4_1():
	con=None
	try:
		#STORAGE
		con=MongoClient("mongodb://localhost:27017")
		db=con["ssnov22"]
		coll=db["employee"]
		try:
			eno=int(dw_ent_eno.get())
			if eno<=0:
				raise Exception("id should not be -ve or zero")
		except Exception:
			showerror("issue","Id should not be empty/non integer/-ve/zero")
			return

		count=coll.count_documents({"_id":eno})
		if count==1:
			coll.delete_one({"_id":eno})
			showinfo("Success","record Deleted")

		else:
			showerror("issue","record does not exists")
	except Exception as e:
		showerror("issue",str(e))
	finally:
		if con is not None:
			con.close()
		dw_ent_eno.delete(0,END)
		dw_ent_eno.focus()

#delete to main
def f4_2():
	dw.withdraw()
	mw.deiconify()

#main to charts window
def f5():
	mw.withdraw()
	cw.deiconify()
	cw_st_data.delete(1.0,END)
	con=None
	try:
		#STORAGE
		con=MongoClient("mongodb://localhost:27017")
		db=con["ssnov22"]
		coll=db["employee"]
		data=coll.find().sort("salary",-1).limit(5)
		name=[]
		salary=[]
		for d in data:
			name.append(str(d["name"]))
			salary.append(float(d["salary"]))
		plt.bar(name,salary)
		plt.xlabel("emp_names")
		plt.ylabel("emp_salary")
		plt.title("Top Employees Name Salary Chart")
		plt.show()
		
		#cw_st_data.insert(INSERT,chart)
	except Exception as e:
		showerror("issue",str(e))
	finally:
		if con is not None:
			con.close()

#chart to main window
def f5_1():
	cw.withdraw()
	mw.deiconify()


#FRONTEND

#main window
mw=Tk()
mw.iconbitmap("Employees.ico")
mw.configure(bg="lightgreen")
mw.title("Employees data app")
mw.geometry("800x800+50+50")
f=("Arial",30,"bold")

mw_btn_add=Button(mw,text="Add",font=f,width=10,command=f1)
mw_btn_view=Button(mw,text="View",font=f,width=10,command=f2)
mw_btn_update=Button(mw,text="Update",font=f,width=10,command=f3)
mw_btn_delete=Button(mw,text="Delete",font=f,width=10,command=f4)
mw_btn_chart=Button(mw,text="charts",font=f,width=10,command=f5)

try:
	urladd="https://ipinfo.io/"
	response=requests.get(urladd)
	data=response.json()
	city=data["city"]
	state=data["region"]
	loctn= "Location : " + str(city) + ", "+ str(state)

	a1 = "https://api.openweathermap.org/data/2.5/weather"
	a2 = "?q=" + city
	a3 = "&appid=" + "c6e315d09197cec231495138183954bd"
	a4 = "&units=" + "metric"
	urladd = a1 + a2 + a3 + a4
	response=requests.get(urladd)
	data=response.json()
	temp=data["main"]["temp"]
	tempt="Temperature : " + str(temp)

except Exception as e:
	loctn="Location : Not Found "
	tempt="Temperature : Not Found"

mw_lab_loc=Label(mw,text=loctn,font=f,bg="lightgreen")

mw_lab_temp=Label(mw,text=tempt,font=f,bg="lightgreen")

mw_btn_add.pack(pady=10)
mw_btn_view.pack(pady=10)
mw_btn_update.pack(pady=10)
mw_btn_delete.pack(pady=10)
mw_btn_chart.pack(pady=10)
mw_lab_loc.place(x=50,y=510)
mw_lab_temp.place(x=50,y=610)

#add window
aw=Toplevel(mw)
aw.iconbitmap("Employees.ico")
aw.configure(bg="lightblue")
aw.title("Add employee data")
aw.geometry("800x800+50+50")

aw_lab_eno=Label(aw,text="enter employee ID ",font=f,bg="lightblue")
aw_ent_eno=Entry(aw,font=f)

aw_lab_name=Label(aw,text="enter name",font=f,bg="lightblue")
aw_ent_name=Entry(aw,font=f)

aw_lab_salary=Label(aw,text="enter salary",font=f,bg="lightblue")
aw_ent_salary=Entry(aw,font=f)

aw_btn_add=Button(aw,text="Add",font=f,width=10,command=f1_1)
aw_btn_back=Button(aw,text="Back",font=f,width=10,command=f1_2)

aw_lab_eno.pack(pady=10)
aw_ent_eno.pack(pady=10)
aw_lab_name.pack(pady=10)
aw_ent_name.pack(pady=10)
aw_lab_salary.pack(pady=10)
aw_ent_salary.pack(pady=10)
aw_btn_add.pack(pady=10)
aw_btn_back.pack(pady=10)
aw.withdraw()

#view window
vw=Toplevel(mw)
vw.iconbitmap("Employees.ico")
vw.configure(bg="lightcyan")
vw.title("View Employee data")
vw.geometry("800x800+50+50")

vw_st_data=ScrolledText(vw,font=f,width=35,height=9)
vw_btn_back=Button(vw,text="Back",font=f,command=f2_1)
vw_st_data.pack(pady=10)
vw_btn_back.pack(pady=10)
vw.withdraw()

#update window
uw=Toplevel(mw)
uw.iconbitmap("Employees.ico")
uw.configure(bg="lightyellow")
uw.title("Update employee data")
uw.geometry("800x800+50+50")

uw_lab_eno=Label(uw,text="enter employee ID ",font=f,bg="lightyellow")
uw_ent_eno=Entry(uw,font=f)

uw_lab_name=Label(uw,text="enter name",font=f,bg="lightyellow")
uw_ent_name=Entry(uw,font=f)

uw_lab_salary=Label(uw,text="enter salary",font=f,bg="lightyellow")
uw_ent_salary=Entry(uw,font=f)

uw_btn_update=Button(uw,text="update",font=f,width=10,command=f3_1)
uw_btn_back=Button(uw,text="Back",font=f,width=10,command=f3_2)

uw_lab_eno.pack(pady=10)
uw_ent_eno.pack(pady=10)
uw_lab_name.pack(pady=10)
uw_ent_name.pack(pady=10)
uw_lab_salary.pack(pady=10)
uw_ent_salary.pack(pady=10)
uw_btn_update.pack(pady=10)
uw_btn_back.pack(pady=10)
uw.withdraw()

#delete window
dw=Toplevel(mw)
dw.iconbitmap("Employees.ico")
dw.configure(bg="lightpink")
dw.title("Delete employee data")
dw.geometry("800x800+50+50")

dw_lab_eno=Label(dw,text="enter employee ID ",font=f,bg="lightpink")
dw_ent_eno=Entry(dw,font=f)

dw_lab_eno.pack(pady=10)
dw_ent_eno.pack(pady=10)

dw_btn_delete=Button(dw,text="Delete",font=f,width=10,command=f4_1)
dw_btn_back=Button(dw,text="Back",font=f,width=10,command=f4_2)

dw_btn_delete.pack(pady=10)
dw_btn_back.pack(pady=10)
dw.withdraw()

#charts window
cw=Toplevel(mw)
cw.iconbitmap("Employees.ico")
cw.configure(bg="lightgrey")
cw.title("View Top Employee charts")
cw.geometry("800x800+50+50")

cw_st_data=ScrolledText(cw,font=f,width=35,height=12)
cw_btn_back=Button(cw,text="Back",font=f,command=f5_1)

cw_st_data.pack(pady=10)
cw_btn_back.pack(pady=10)
cw.withdraw()

mw.mainloop()


```

# **Internship Progress Report**

# Contents
  - [Read Me](#read-me)
  - [Tasks Performed](#task-performed)
  - [Notes](#notes)
  
# Read Me
- This is a progress report on what have been done during internship period.
- All codes were compiled and run using **Visual** Code studio.
- The operating system use is **Ubuntu** operating system.
  
# Tasks Performed

## **Python and Linux**

- First week is to learn and explore python3 and linux. Create directory using *mkdir* command, show list of file using *ls* command and change to another directory using *cd* command. Another commands are to create file using *touch* or *cat* command, to remove directory using *rmdir* command and any other commands.

## **Git and GitHub**

- An introduction using [Git](https://git-scm.com/) and [GitHub](https://github.com/). GitHub is web-based hosting service for version control. Issue the command of installing the git ```sudo apt install git-all``` in terminal window. To add file into the project, issue the command ```git add -name of the file-``` and check the file by issuing ```git status``` command to check the status of git.

- To commit all changes that have been made in a file, issue the command ```git commit -m "message"``` where message is your message about the changes. Branch is to separate development work without affecting other branches. Create new branch ```git checkout -b EXPERIMENT``` where **EXPERIMENT** is the name of new branch. To push project into GitHub repository ```git push -u origin master ```.

## **Task 1**

- Task 1 is to retrieve the McDonald's stores json data from [McDonald's wesite](https://www.mcdonalds.com.my/locate-us).

### **Import library** 

- First task is to store json data of [McDonald's](https://www.mcdonalds.com.my/locate-us) into sqlite3 database. Three library need to be imported which are *requests*, *json* and *sqlite3*. These three are the libraries of Python.

```python
import requests
import json
import sqlite3
```

### **POST requests**

- POST method sends data to the server. POST method is to submit data to be processed to the server. Allows to send data to the request by passing **dictionary** to the **data**.

```python
data = {
    'ajax': '1',
    'action': 'get_nearby_stores',
    'distance': '100',
    'lat': '4.210484',
    'lng': '101.975',
    'state': '',
    'products': '',
    'address': '',
    'issuggestion': '0',
    'islocateus': '0'
}
resp = requests.post(endpoint, data=data, headers=headers)
```

- The **data** contains the important keys and values that are needed for getting the JSON data.

### **Encoding**

- The code below set the encoding of the content to be **'utf-8-sig'**. Without setting the encoding to 'utf-8-sig', error will raise.

```python
resp.encoding = 'utf-8-sig'
```

### **Defining the api-endpoint**

```python
endpoint = 'https://www.mcdonalds.com.my/storefinder/index.php'
```

- The **headers** is passed to the **headers** parameter in a request as shown in below.

```python
resp = requests.post(endpoint, data=data, headers=headers)
```

- Now, it has a sending **post** request and object called **resp**. We can get all the information we need from this object. Use *requests.post()* method since it sends a **POST** request.

### **JSON data to a Python object**

- For **json string**, use *json.loads()* method to parse it. Result will be in **Python dictionary**.

```python
stores = json.loads(resp.text)
```

### **Create database SQLite for storing data**

- Create database to store data of McDonald's and open a connection to a database.

```python
conn=sqlite3.connect("mcdonalds.db")
c=conn.cursor()
```

- After that, create a table **stores** for storing *McDonald's store* information.

**CREATE TABLE** statement is used by calling **execute()** statement of a cursor.

```python
c.execute("""CREATE TABLE IF NOT EXISTS stores (storeName TEXT, address TEXT, telephone TEXT, email TEXT, website TEXT, fax TEXT, desc TEXT, lat REAL, long REAL, cat TEXT)""")
```

- To store information to **SQLite** database, **INSERT** statement is executed by calling **execute()** statement of the cursor. **For** loop is used to insert each information into database.

```python
for s in stores ["stores"]:
    list = []
    for cat in s["cat"]:
         list.append(cat["cat_name"])
       # print(cat["cat_name"])
    p = json.dumps(list)
    #print(p) 

    c.execute("INSERT OR IGNORE INTO stores VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)",
    (s["name"],s["address"],s["telephone"],s["email"],s["website"],s["fax"], s["description"], s["lat"], s["lng"], p))
```

## **Task 2**

### **Web Scraping 1**

- Web scrape is to extract desired information from a webpage. The two websites are [GDACS](http://www.gdacs.org/) and [Natural Disaster RSS Feed](https://www.arcgis.com/home/item.html?id=6e331edd00dc48a88a4a4fa1dd5cff2a). 

- There are three libraries need to be imported which are *requests*, *json* and *sqlite3*. 

- For GDACS, *get* method is used. *get* method is to get or retrieve the data from the server. *resp* object is created which will store the request response. Used *requests.get()* since **GET** method is used. The request URL of the website can be obtained in headers. Since GDACS is a *json* data, there is no need to import another library.

```python
endpoint = 'http://www.gdacs.org/xml/gdacsDR.geojson'
resp = requests.get(endpoint)
```

- For **json string**, use *json.loads()* method to parse it. Result will be in **Python dictionary**.

```python
droughts = json.loads(resp.text)
```

- Create a database to store the useful information that can get from the website and open a connection to database. 

```python
conn=sqlite3.connect("gdacs.db")
c=conn.cursor()
```

- After that, create a table for storing the useful information.
**CREATE TABLE** statement is used by calling **execute()** statement of a cursor.

```python
c.execute("""CREATE TABLE IF NOT EXISTS droughts (id TEXT, description TEXT, htmldescription TEXT, alertlevel REAL, 
alertscore REAL, eventtype TEXT, eventid TEXT, episodeid TEXT, fromdate TEXT, todate TEXT, country TEXT, severity REAL, latitude REAL,
longitude REAL, polygonlabel TEXT, polygontype TEXT, Class TEXT, coordinates REAL)""")
```
- To store information to **SQLite** database, **INSERT** statement is executed by calling **execute()** statement of the cursor. **For** loop is used to insert each information into database.

```python
for d in droughts['features']:
    list = []

    for coordinate in d['geometry']['coordinates']:
        # list = []
        list.append(coordinate)
        # print(list)
    p = json.dumps(list)

    alertscore = d['properties'].get('alertscore')
    eventid = d['properties'].get('eventid')
    episodeid = d['properties'].get('episodeid')
    polygontype = d['properties'].get('polygontype')

    c.execute("INSERT OR IGNORE INTO droughts VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)", 
    (d['id'], d['properties']['description'], d['properties']['htmldescription'], 
    d['properties']['alertlevel'], alertscore, d['properties']['eventtype'], eventid, episodeid, d['properties']['fromdate'], 
    d['properties']['todate'], d['properties']['country'], d['properties']['severity'], d['properties']['latitude'], 
    d['properties']['longitude'], d['properties']['polygonlabel'], polygontype, d['properties']['Class'], p))
```
- In this case, some of the list do not have key values, so to get the information, *get* method is used. 

- To view the information in the database, go to terminal and type **sqlite3 gdacs.db** and select any information that you want to view.

### **Web Scraping 2**

- For Natural Disaster RSS Feed, there are four libraries need to be imported which are *requests*, *json*, *sqlite3*, and *BeautifulSoup* since the website has no *json* data but HTML. **BeautifulSoup** is a library that scrape information especially from **HTML** and **XML** files.

- 

## **HTML, CSS and Javascript**

## **Task 3**

- First, is to learn HTML. HTML is Hypertext Markup Language that defines meaning and structure of web content. Example of HTML is as given below:

```<p>Hello, world!</p>```

- It consists opening and closing tag. Opening tag is ```<p>``` and closing tag is ```</p>```.

- First, is to create a portfolio page using HTML and CSS using [Bootstrap](https://getbootstrap.com/) Bootstrap is a framework for building responsive and mobile-first sites. Cascading Style Sheets (CSS) is how elements are displayed on the screens. Image shown below is a Home page of portfolio page using Materialize CSS, source: [Portfolio](https://www.youtube.com/watch?v=HcSLoR7oGHU)

![alt text](image.png "Portfolio: Home page")

- The header is an image style with CSS. Side of the portfolio page is navigation bar section with Home, About and Contact. On the right section is paragraph with ```<p>``` tag. The paragraph is syle with;

```article {
float: left;
font: Verdana;
text-align: justify;
letter-spacing: 1px;
line-height: 1.8;
font-size: 13px;
padding: 20px;
width: 70%;
background-color: #f1f1f1;
height: 300px;
}
```

- Image shown below is About page. About page is just the same with Home page. Header with an image style with CSS, left side is navigation bar and right side is content.

![alt text](image_1.png "Portfolio: About page")

- Image shown below is Contact page. Contact page also is just the same with Home and About page. Header with an image style with CSS, left side is navigation bar that can go back to Home or About page if click on it. Contact page also have social media that will be directed to the page if click on it.

![alt text](image_2.png "Portfolio: Contact page")

## **Task 4**

- Next is to learn HTML, CSS and Javascript. Javascript is a computer programming language to create interactive effects with web browsers. Second is, create travel agency (source:) [Youtube](https://www.youtube.com/watch?v=MaP3vO-vEsg) by following tutorial on youtube.

- Top of the website is a fixed navigation bar with navigations links.
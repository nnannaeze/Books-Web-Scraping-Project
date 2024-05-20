#  Web Scraping Project on a Books Website
## Problem Statement:

The objective of this web scraping project is to develop a robust and efficient tool capable of extracting key information from a book retailer’s website. The tool will systematically navigate through the website, accurately identifying and collecting data on book titles, prices, stock availability, and hyperlinks to individual book pages. This data will be meticulously parsed and structured to ensure consistency and readability.

Upon successful extraction, the tool will then proceed to convert the gathered information into a well-organized and formatted CSV or Excel file. This file will serve as a comprehensive database, enabling easy access and analysis of the book inventory for various stakeholders, such as consumers, analysts, and other interested parties.

The tool will be designed with a focus on adaptability to handle potential changes in the website’s layout and ensure compliance with web scraping best practices and legal guidelines. The end result will be a streamlined process that facilitates the monitoring of book-related data, supports price comparison, and aids in inventory management.
### Key Deliverables:
- A web scraping tool that navigates and extracts data from a book website.
- Extraction of book titles, prices, stock status, and URLs.
- Conversion of the scraped data into a CSV or Excel file for easy analysis.
- Adherence to ethical scraping guidelines and respect for the website's terms of service.

 ## Preliminary: Importing libraries and defining known values

```python

from bs4 import BeautifulSoup
import requests
import pandas as pd
import numpy as np
import sqlite3
from datetime import datetime

```


url = "https://books.toscrape.com/catalogue/page-1.html"
![image](https://github.com/nnannaeze/Books-Web-Scraping-Project/assets/148848746/53ebecba-d8b4-41d2-886b-e75ef00f742d)

![image](https://github.com/nnannaeze/Books-Web-Scraping-Project/assets/148848746/3f5a2358-7771-4bc1-8415-219cef4f1319)

page = requests.get(url)
soup = BeautifulSoup(page.text,"html.parser"


the above result is correct because if we go to the any white space in the web page and rightclick, select **view page source**

we will be pushed to a new tab and we'll see that the Title attribute = 
![image](https://github.com/nnannaeze/Books-Web-Scraping-Project/assets/148848746/992f5ec2-37f7-4ef0-8425-45dea08b9194)

We want to scrape to the end of the pages = 50
assuming we do not know where the pages stop

but we know that once the pages stop like if we go and paste 100 in the page number in the url(https://books.toscrape.com/catalogue/page-100.html) 

we will get **404 Not Found**
and if we right click on the open space and select **view page source**, we will see this

![image](https://github.com/nnannaeze/Books-Web-Scraping-Project/assets/148848746/db98e321-2bf1-4e10-ae10-6b9ae6b59382)

## The Stages for the body of the code neccessary for effective web scraping
### **Task 1**
### *State the knowns*

we have more than 1 page to scrape and each page contains more than 1 item to scrape therefore we need our url to be dynamic

i.e change as the pages change on tha note we'll start with creating a dynamic **url**

url = "https://books.toscrape.com/catalogue/page-1.html"

To make this url dynamic , we'll replace the `page-1` with `page-current page` therefore we'll introduce the variable `current_page`

```python
current_page = 1

url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
page = requests.get(url)
soup = BeautifulSoup(page.text,"html.parser")

```
    

### **Task 2**
### *Identify/Introduce the Loops*
1. We're scraping more than 1 page and in each page we have more than an Item to scrape
2. We'll want the computer to stop at the end of the pages and we already know that  after the last valid page the rest invalid pages will show Title = `404 Not Found`
3. The above fact will serve as the deciding factor for terminating the loop.
enhancing our code we add

```python
current_page = 1

url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
page = requests.get(url)
soup = BeautifulSoup(page.text,"html.parser")
if soup.title.text == '404 Not Found':
    proceed = False
else:
```

We'll need to add `proceed = True` and also add `while(proceed == true):` since we've added `proceed = False` and observe its due **identations**
```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
```


### **Task 3**
### *Extracting information*

1. Go to the web page and select the attribute that encompasses the whole object from where we'll be scraping using the command `find_all`
2. determine items to scrape in our case
    * `Title`
    * `Link`
    * `Price`
    * `Stock`
3. Create a dictionary `item` where this informations will be stored
4. Note that to know what to scrape, you need to rightclick on the the item asnd then select `inspect` hover on the html and notice which line corresponds ur item of choice, use `book.find().atrrs[]` or `book.find("p",class_="") for p class
5. Insert a counter that will move the iteration to the next page current_page +=1
```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color')
            item['In_Stock'] = book.find('p',class_='instock availability')
        
        
        current_page += 1
```


### **Task 4**
### *Validation*
1. we'll check using the print statement to validate how well the scraping worked
2. we'll introduce a terminator `proceed = False` to ensure the check is for just the current page

```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color')
            item['In_Stock'] = book.find('p',class_='instock availability')
        
            print(item['Title'])
            proceed = False
            
        current_page += 1
```
Output 

```
A Light in the Attic
Tipping the Velvet
Soumission
Sharp Objects
Sapiens: A Brief History of Humankind
The Requiem Red
The Dirty Little Secrets of Getting Your Dream Job
The Coming Woman: A Novel Based on the Life of the Infamous Feminist, Victoria Woodhull
The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics
The Black Maria
Starving Hearts (Triangular Trade Trilogy, #1)
Shakespeare's Sonnets
Set Me Free
Scott Pilgrim's Precious Little Life (Scott Pilgrim #1)
Rip it Up and Start Again
Our Band Could Be Your Life: Scenes from the American Indie Underground, 1981-1991
Olio
Mesaerion: The Best Science Fiction Stories 1800-1849
Libertarianism for Beginners
It's Only the Himalayas
```

**Good work!!**


***2. Lets validate the `Link`***
current_page = 1

```python
proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color')
            item['In_Stock'] = book.find('p',class_='instock availability')
        
            print(item['Link'])
            proceed = False
            
        current_page += 1
```
Output
```
a-light-in-the-attic_1000/index.html
tipping-the-velvet_999/index.html
soumission_998/index.html
sharp-objects_997/index.html
sapiens-a-brief-history-of-humankind_996/index.html
the-requiem-red_995/index.html
the-dirty-little-secrets-of-getting-your-dream-job_994/index.html
the-coming-woman-a-novel-based-on-the-life-of-the-infamous-feminist-victoria-woodhull_993/index.html
the-boys-in-the-boat-nine-americans-and-their-epic-quest-for-gold-at-the-1936-berlin-olympics_992/index.html
the-black-maria_991/index.html
starving-hearts-triangular-trade-trilogy-1_990/index.html
shakespeares-sonnets_989/index.html
set-me-free_988/index.html
scott-pilgrims-precious-little-life-scott-pilgrim-1_987/index.html
rip-it-up-and-start-again_986/index.html
our-band-could-be-your-life-scenes-from-the-american-indie-underground-1981-1991_985/index.html
olio_984/index.html
mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html
libertarianism-for-beginners_982/index.html
its-only-the-himalayas_981/index.html
```

**Good but didnt appear as an absolute link(relative Link)**

The  item['Link'] shows relative values, to make it show absolute link value.

to achieve this we go and copy `"https://books.toscrape.com/catalogue/"` and position it before the `book.find("a")`

to look like this `item['Link'] = "https://books.toscrape.com/catalogue/page-"+book.find("a").attrs["href"]`

```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = "https://books.toscrape.com/catalogue/"+book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color')
            item['In_Stock'] = book.find('p',class_='instock availability')
        
            print(item['Link'])
            proceed = False
            
        current_page += 1
```
Output
```
https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html
https://books.toscrape.com/catalogue/tipping-the-velvet_999/index.html
https://books.toscrape.com/catalogue/soumission_998/index.html
https://books.toscrape.com/catalogue/sharp-objects_997/index.html
https://books.toscrape.com/catalogue/sapiens-a-brief-history-of-humankind_996/index.html
https://books.toscrape.com/catalogue/the-requiem-red_995/index.html
https://books.toscrape.com/catalogue/the-dirty-little-secrets-of-getting-your-dream-job_994/index.html
https://books.toscrape.com/catalogue/the-coming-woman-a-novel-based-on-the-life-of-the-infamous-feminist-victoria-woodhull_993/index.html
https://books.toscrape.com/catalogue/the-boys-in-the-boat-nine-americans-and-their-epic-quest-for-gold-at-the-1936-berlin-olympics_992/index.html
https://books.toscrape.com/catalogue/the-black-maria_991/index.html
https://books.toscrape.com/catalogue/starving-hearts-triangular-trade-trilogy-1_990/index.html
https://books.toscrape.com/catalogue/shakespeares-sonnets_989/index.html
https://books.toscrape.com/catalogue/set-me-free_988/index.html
https://books.toscrape.com/catalogue/scott-pilgrims-precious-little-life-scott-pilgrim-1_987/index.html
https://books.toscrape.com/catalogue/rip-it-up-and-start-again_986/index.html
https://books.toscrape.com/catalogue/our-band-could-be-your-life-scenes-from-the-american-indie-underground-1981-1991_985/index.html
https://books.toscrape.com/catalogue/olio_984/index.html
https://books.toscrape.com/catalogue/mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html
https://books.toscrape.com/catalogue/libertarianism-for-beginners_982/index.html
https://books.toscrape.com/catalogue/its-only-the-himalayas_981/index.html
```
**Good work!!**

***3. Lets validate the `Price`***

```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = "https://books.toscrape.com/catalogue/"+book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color')
            item['In_Stock'] = book.find('p',class_='instock availability')
        
            print(item['Price'])
            proceed = False
            
        current_page += 1
```
Output
```
<p class="price_color">Â£51.77</p>
<p class="price_color">Â£53.74</p>
<p class="price_color">Â£50.10</p>
<p class="price_color">Â£47.82</p>
<p class="price_color">Â£54.23</p>
<p class="price_color">Â£22.65</p>
<p class="price_color">Â£33.34</p>
<p class="price_color">Â£17.93</p>
<p class="price_color">Â£22.60</p>
<p class="price_color">Â£52.15</p>
<p class="price_color">Â£13.99</p>
<p class="price_color">Â£20.66</p>
<p class="price_color">Â£17.46</p>
<p class="price_color">Â£52.29</p>
<p class="price_color">Â£35.02</p>
<p class="price_color">Â£57.25</p>
<p class="price_color">Â£23.88</p>
<p class="price_color">Â£37.59</p>
<p class="price_color">Â£51.33</p>
<p class="price_color">Â£45.17</p>
```
**Observation** :
Good but we want only the number(figures) appearing
1. we add `.text` at the end of the `price` expression
code

```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = "https://books.toscrape.com/catalogue/"+book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color').text
            item['In_Stock'] = book.find('p',class_='instock availability')
        
            print(item['Price'])
            proceed = False
            
        current_page += 1
```
Output
```
Â£51.77
Â£53.74
Â£50.10
Â£47.82
Â£54.23
Â£22.65
Â£33.34
Â£17.93
Â£22.60
Â£52.15
Â£13.99
Â£20.66
Â£17.46
Â£52.29
Â£35.02
Â£57.25
Â£23.88
Â£37.59
Â£51.33
Â£45.17
```
**Observation** :
Good but we want only the number(figures) appearing
1. we'll use slicers to start after the 1st 2 index `[2:]`

```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = "https://books.toscrape.com/catalogue/"+book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color').text[2:]
            item['In_Stock'] = book.find('p',class_='instock availability')
        
            print(item['Price'])
            proceed = False
            
        current_page += 1
```
Output
```
51.77
53.74
50.10
47.82
54.23
22.65
33.34
17.93
22.60
52.15
13.99
20.66
17.46
52.29
35.02
57.25
23.88
37.59
51.33
45.17
```
**Observation** :

Good work!!!


***4. Lets validate the `in_stock`***

```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = "https://books.toscrape.com/catalogue/"+book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color').text[2:]
            item['In_Stock'] = book.find('p',class_='instock availability')
        
            print(item['In_Stock'])
            proceed = False
            
        current_page += 1
```

Output
```
In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
<p class="instock availability">
<i class="icon-ok"></i>
    
        In stock
    
</p>
```
**Observation** :
1. We should always include `.text` at end of every `"p",class_=` to stop repeating introduction
2. Include `strip()` to trim the wide spaces

```python
current_page = 1

proceed = True
while(proceed):    #this is same with while(proceed==True)
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = "https://books.toscrape.com/catalogue/"+book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color').text[2:]
            item['In_Stock'] = book.find('p',class_='instock availability').text.strip()
        
            print(item['In_Stock'])
            proceed = False
            
        current_page += 1
```
Output
```
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
In stock
```
**Observation** :
Goodwork!!!

#### Task 5
**Loading**
1. Remove the Terminator `proceed = True)` and the Print() statements
2. To effectively load this data `item` into a CSV or Excel file we'll have to convert the dictionary `item` into a list `data`
3. we'll introduce an empty list `data[]` and make sure that after every iteration for each item, they are stored in the data list
4. Lets log the process so that while the computer scraping we'll see the current page
5. Convert to dataframe `df` using the pandas library
6. Load df to `CSV` and Load df to Excel


```python
current_page = 1

data = []
proceed = True
while(proceed):    #this is same with while(proceed==True)
    print("currently Scraping Page :"+str(current_page))
    url = "https://books.toscrape.com/catalogue/page-"+str(current_page)+".html"
    page = requests.get(url)
    soup = BeautifulSoup(page.text,"html.parser")
    if soup.title.text == '404 Not Found':
        proceed = False
    else:
        all_books = soup.find_all("li",class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
        for book in all_books:
            item = {}
            item['Title'] = book.find("img").attrs["alt"]
            item['Link'] = "https://books.toscrape.com/catalogue/"+book.find("a").attrs["href"]
            item['Price'] = book.find('p',class_='price_color').text[2:]
            item['In_Stock'] = book.find('p',class_='instock availability').text.strip()
        
            data.append(item)
         
            
        current_page += 1
df = pd.DataFrame(data)      
df.to_excel("books.xlsx")    
df.to_csv("books.csv")
```
Output
```
currently Scraping Page :1
currently Scraping Page :2
currently Scraping Page :3
currently Scraping Page :4
currently Scraping Page :5
currently Scraping Page :6
currently Scraping Page :7
currently Scraping Page :8
currently Scraping Page :9
currently Scraping Page :10
currently Scraping Page :11
currently Scraping Page :12
currently Scraping Page :13
currently Scraping Page :14
currently Scraping Page :15
currently Scraping Page :16
currently Scraping Page :17
currently Scraping Page :18
currently Scraping Page :19
currently Scraping Page :20
currently Scraping Page :21
currently Scraping Page :22
currently Scraping Page :23
currently Scraping Page :24
currently Scraping Page :25
currently Scraping Page :26
currently Scraping Page :27
currently Scraping Page :28
currently Scraping Page :29
currently Scraping Page :30
currently Scraping Page :31
currently Scraping Page :32
currently Scraping Page :33
currently Scraping Page :34
currently Scraping Page :35
currently Scraping Page :36
currently Scraping Page :37
currently Scraping Page :38
currently Scraping Page :39
currently Scraping Page :40
currently Scraping Page :41
currently Scraping Page :42
currently Scraping Page :43
currently Scraping Page :44
currently Scraping Page :45
currently Scraping Page :46
currently Scraping Page :47
currently Scraping Page :48
currently Scraping Page :49
currently Scraping Page :50
currently Scraping Page :51
```

#### Task 6
**Load CSV**

```python
df = pd.read_csv("books.csv")
df.head()
```
Output
![image](https://github.com/nnannaeze/Books-Web-Scraping-Project/assets/148848746/898f2902-c5ed-40d8-9998-6f09e46389ed)

To check for the size and shape of the dataset
```python
df.size
```
Output
```
5000
```
```python
df.shape
```
Output
```
(1000, 5)
```
#### Excel Format
![image](https://github.com/nnannaeze/Books-Web-Scraping-Project/assets/148848746/9e15080d-018d-42ed-87ba-7f150091bf7b)
## Conclusion:

The web scraping project aimed at extracting critical information from a book retailer’s website has been successfully completed. The tool developed for this purpose has proven to be highly effective in navigating the website, identifying, and retrieving data on book titles, prices, stock levels, and URLs. The accuracy and reliability of the data collected are commendable, reflecting the tool’s robustness against potential challenges such as website layout changes.

The conversion process, which reformats the scraped data into a CSV or Excel file, has been executed flawlessly, resulting in a well-structured and comprehensive dataset. This dataset not only enhances the accessibility and usability of the information but also opens avenues for in-depth analysis and strategic decision-making.

This project has demonstrated the power of web scraping in transforming raw website data into valuable insights. The success of this endeavor underscores the potential for similar tools to revolutionize data collection and analysis across various domains. As we conclude, we celebrate the seamless integration of technology and data management that this project represents.

### Key Achievements:
- Successful extraction of book-related data from a web source.
- Accurate parsing and structuring of information into a user-friendly format.
- Creation of a versatile dataset for analysis and inventory management.
- Demonstration of ethical web scraping practices and compliance with legal standards.


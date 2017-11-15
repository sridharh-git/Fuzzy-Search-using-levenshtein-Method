

```python
from fuzzywuzzy import process
import time
```

Function lookupStr takes a semi-colon separated provider row and returns Key to use for dict.
Key = first Char Last Name + First Char First Name
Ex: 1962405183;FOX;DANIEL;2240 SUTHERLAND AVE;STE 107;KNOXVILLE;TN;379192333;2085R0202X ==> FD


```python
def lookupStr(line):
    splitLine = line.split(";")
    lastName = splitLine[1][0] if splitLine[1] != '' else ''
    firstName = splitLine[2][0] if splitLine[2] != '' else ''
    return lastName + firstName
```

Create  dict to store provider data from file. 
If Key already exists append provider record to existing list else create new one.


```python
npiLookup = {}

#f = open("C:\\Users\\sridharh\\Desktop\\Temp\\NPI_Extract.dat")
f = open("NPI_Extract.dat")
lines = f.readlines()

for line in lines:
    dictLookup = lookupStr(line)
    if(dictLookup in npiLookup.keys()):
        currList = npiLookup[dictLookup]
        currList.append(line)
        npiLookup[dictLookup] = currList
    else:
        npiLookup[dictLookup] = [line]
```

Function getSuggestion takes input string and returns first closest match from the provider records.


```python
def getSuggestion(provString):
    lkpStr=lookupStr(provString)
    if lkpStr in npiLookup : # if lookup exists
        return process.extract(provString, npiLookup[lkpStr], limit=1)[0][0].split(";") , len(npiLookup[lkpStr])
    
    elif len(lkpStr) == 1: # if lookup does not exists and either first name or last name missing
        mergList = []
        for key in npiLookup:
            if((lkpStr) in key): #, search for all keys contains lookup string
                for lst in npiLookup[key]:
                    mergList.append(lst)
        rtrnString = [process.extract(provString, mergList, limit=1)[0][0].split(";"), len(mergList)]
        mergList=[]
        return rtrnString
    
    else:  # if both first name and last name missing, or lookup value not in key 
        return ["Full scan require to find best match!", 0]
```

Testing....


```python
choices = ["1508860545;KOLTIS;GORDON;2234 COLONIAL BLVD;MANAGED CARE DEPT;FORT MYERS;FL;339071412;2085R0001X",
           ";ZOLMAN;MARK;7910 W JEFFERSON BLVD;SUITE 300;FORT WAYNE;IN;468044159;208100000X",
           "1659375996;CR;WILLIAM;PO BOX 650865;;DALLAS;TX;75265065;207L00000X",
           "1508545;KOLTIS;GORDON;2234 COLONIAL BLVD;MANAGED CARE DEPARTMENT;FORT MYERS;FL;339071412;2085R0001X",
           "150886;KOLTIS;;2234 COLONIAL BLVD;MANAGED CARE DEPT;FORT MYERS;FL;339071412;2085R0001X",
           ";;GORDON;2234 COLONIAL ;MANAGED CARE DEPPT;FORT MYERS;FL;339071412;2085R0001X",
           ";KLTIS;GODN;2234 COLONIAL BLVD;;FT MYERS;FL;33907;2085R0001X",
           ";;;2234 COLONIAL BLVD;;FT MYERS;FL;33907;2085R0001X"
          ]

```


```python
for testItem in choices:
    print("\n")
    strtTime = time.time()
    print(testItem.split(";"))
    reslt = getSuggestion(testItem)
    print(reslt[0])
    print("Total Scanned Items : " + str(format(reslt[1],',') ))
    print("Execution Time : " + str(round(time.time() - strtTime, 2)) + " Secs" )

```

    
    
    ['1508860545', 'KOLTIS', 'GORDON', '2234 COLONIAL BLVD', 'MANAGED CARE DEPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X']
    ['1508860545', 'KOLTIS', 'GORDON', '2234 COLONIAL BLVD', 'MANAGED CARE DEPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X\n']
    Total Scanned Items : 3,807
    Execution Time : 0.56 Secs
    
    
    ['', 'ZOLMAN', 'MARK', '7910 W JEFFERSON BLVD', 'SUITE 300', 'FORT WAYNE', 'IN', '468044159', '208100000X']
    ['1467455436', 'ZOLMAN', 'MARK', '7910 W JEFFERSON BLVD', 'SUITE 300', 'FORT WAYNE', 'IN', '468044159', '208100000X\n']
    Total Scanned Items : 2,675
    Execution Time : 0.35 Secs
    
    
    ['1659375996', 'CR', 'WILLIAM', 'PO BOX 650865', '', 'DALLAS', 'TX', '75265065', '207L00000X']
    ['1659375996', 'CONNER', 'WILLIAM', 'PO BOX 650865', '', 'DALLAS', 'TX', '752650865', '207L00000X\n']
    Total Scanned Items : 3,904
    Execution Time : 0.5 Secs
    
    
    ['1508545', 'KOLTIS', 'GORDON', '2234 COLONIAL BLVD', 'MANAGED CARE DEPARTMENT', 'FORT MYERS', 'FL', '339071412', '2085R0001X']
    ['1508860545', 'KOLTIS', 'GORDON', '2234 COLONIAL BLVD', 'MANAGED CARE DEPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X\n']
    Total Scanned Items : 3,807
    Execution Time : 0.59 Secs
    
    
    ['150886', 'KOLTIS', '', '2234 COLONIAL BLVD', 'MANAGED CARE DEPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X']
    ['1508860545', 'KOLTIS', 'GORDON', '2234 COLONIAL BLVD', 'MANAGED CARE DEPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X\n']
    Total Scanned Items : 313,763
    Execution Time : 37.67 Secs
    
    
    ['', '', 'GORDON', '2234 COLONIAL ', 'MANAGED CARE DEPPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X']
    ['1508860545', 'KOLTIS', 'GORDON', '2234 COLONIAL BLVD', 'MANAGED CARE DEPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X\n']
    Total Scanned Items : 230,702
    Execution Time : 28.67 Secs
    
    
    ['', 'KLTIS', 'GODN', '2234 COLONIAL BLVD', '', 'FT MYERS', 'FL', '33907', '2085R0001X']
    ['1508860545', 'KOLTIS', 'GORDON', '2234 COLONIAL BLVD', 'MANAGED CARE DEPT', 'FORT MYERS', 'FL', '339071412', '2085R0001X\n']
    Total Scanned Items : 3,807
    Execution Time : 0.91 Secs
    
    
    ['', '', '', '2234 COLONIAL BLVD', '', 'FT MYERS', 'FL', '33907', '2085R0001X']
    Full scan require to find best match!
    Total Scanned Items : 0
    Execution Time : 0.0 Secs
    
